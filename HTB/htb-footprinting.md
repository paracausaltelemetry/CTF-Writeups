---
title: Footprinting Lab
summary: Hack The Box Academy footprinting lab focused on careful service enumeration, FTP access, SSH key discovery, and retrieving the proof flag.
date: 2026-04-05
tags: [Hack The Box, Academy, Recon, Footprinting, FTP, SSH]
difficulty: easy
os: Linux
url: https://academy.hackthebox.com/app/module/112/section/1078
---

# CTF Room: Footprinting Lab
- [Link to room](https://academy.hackthebox.com/app/module/112/section/1078)
- **Difficulty:** Easy
- **Category:** Recon

## 1. Brief
We were commissioned by the company `Inlanefreight Ltd` to test three different servers in their internal network. The company uses many different services, and the IT security department felt that a penetration test was necessary to gain insight into their overall security posture.

The first server is an internal DNS server that needs to be investigated. In particular, our client wants to know what information we can get out of these services and how this information could be used against its infrastructure. Our goal is to gather as much information as possible about the server and find ways to use that information against the company. However, our client has made it clear that it is forbidden to attack the services aggressively using exploits, as these services are in production.

Additionally, our teammates have found the following credentials "*ceil:||qwer1234||*", and they pointed out that some of the company's employees were talking about SSH keys on a forum.

The administrators have stored a `flag.txt` file on this server to track our progress and measure success. Fully enumerate the target and submit the contents of this file as proof.

## 2. Questions

#### Enumerate the server carefully and find the flag.txt file. Submit the contents of this file as the answer.

We have been given the credentials ceil:||qwer1234|| and told that we need to look out for SSH keys on a forum. Immediately I think I should be looking for a private key to use with the SSH service.

I began by enumerating the open ports on the machine.
### NMAP Scan
```bash
nmap -sV -sC --top-ports 1000 footprinting.htb
```
```bash
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-10-31 14:12 IST Nmap scan report for machine_ip Host is up (0.79s latency).
Not shown: 996 closed tcp ports (reset)

PORT STATE SERVICE VERSION
21/tcp open ftp
| fingerprint-strings:
| GenericLines:
| 220 ProFTPD Server (ftp.int.inlanefreight.htb) [footprinting.htb]
| Invalid command: try being more creative
|_ Invalid command: try being more creative
22/tcp open ssh OpenSSH 8.2p1 Ubuntu 4ubuntu0.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
| 3072 3f:4c:8f:10:f1:ae:be:cd:31:24:7c:a1:4e:ab:84:6d (RSA)
| 256 7b:30:37:67:50:b9:ad:91:c0:8f:f7:02:78:3b:7c:02 (ECDSA)
|_ 256 88:9e:0e:07:fe:ca:d0:5c:60:ab:cf:10:99:cd:6c:a7 (ED25519)
53/tcp open domain ISC BIND 9.16.1 (Ubuntu Linux)
| dns-nsid:
|_ bind.version: 9.16.1-Ubuntu 2121/tcp open ftp
| fingerprint-strings:
| GenericLines:
| 220 ProFTPD Server (Ceils FTP) [machine_ip]
| Invalid command: try being more creative
|_ Invalid command: try being more creative

Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

I know that I will not be able to use SSH until I have the mentioned key, so I decided to have a look in the FTP server on port 2121, the "Ceil's FTP" makes me think that I can use the given credentials here.
### FTP Conversation
```bash
ftp footprinting.htb 2121
```
```bash
Connected to machine_ip.
220 ProFTPD Server (Ceils FTP) [machine_ip]
Name (machine_ip:offsec): ceil
331 Password required for ceil
Password:
230 User ceil logged in Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls -la
229 Entering Extended Passive Mode (|\|\|38580|)
150 Opening ASCII mode data connection for file list
drwxr-xr-x 4 ceil ceil 4096 Nov 10 2021 .
drwxr-xr-x 4 ceil ceil 4096 Nov 10 2021 ..
-rw------- 1 ceil ceil 294 Nov 10 2021 .bash_history
-rw-r--r-- 1 ceil ceil 220 Nov 10 2021 .bash_logout
-rw-r--r-- 1 ceil ceil 3771 Nov 10 2021 .bashrc
drwx------ 2 ceil ceil 4096 Nov 10 2021 .cache
-rw-r--r-- 1 ceil ceil 807 Nov 10 2021 .profile
drwx------ 2 ceil ceil 4096 Nov 10 2021 .ssh
-rw------- 1 ceil ceil 759 Nov 10 2021 .viminfo
226 Transfer complete
```

Since we are looking for SSH credentials, maybe lets check the .ssh directory (nice one XD). Doing so reveals the SSH key for the user "Ceil". Huzzah!

### FTP further commands
```bash
ftp> ls
229 Entering Extended Passive Mode (|\|\|39113|)
150 Opening ASCII mode data connection for file list
-rw-rw-r--   1 ceil     ceil          738 Nov 10  2021 authorized_keys
-rw-------   1 ceil     ceil         3381 Nov 10  2021 id_rsa
-rw-r--r--   1 ceil     ceil          738 Nov 10  2021 id_rsa.pub
226 Transfer complete
ftp> get id_rsa
local: id_rsa remote: id_rsa
229 Entering Extended Passive Mode (|\|\|41505|)
150 Opening BINARY mode data connection for id_rsa (3381 bytes)
100% |******************************************************************|  3381       10.74 KiB/s    00:00 ETA
226 Transfer complete
3381 bytes received in 00:01 (2.48 KiB/s)
```

Since this is an SSH key, it is important that we use chmod to edit the file's privileges.

### chmod command
```bash
chmod 600 id_rsa
```

### SSH Conversation
```bash
ssh -i id_rsa ceil@footprinting.htb
```
```bash
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'footprinting.htb' (ED25519) to the list of known hosts.
Welcome to Ubuntu 20.04.1 LTS (GNU/Linux 5.4.0-90-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

System information as of Thu 31 Oct 2024 09:14:33 AM UTC

System load:  0.01                Processes:             159
Usage of /:   86.7% of 3.87GB     Users logged in:       0
Memory usage: 12%                 IPv4 address for ens192: machine_ip
Swap usage:   0%

=> / is using 86.7% of 3.87GB

118 updates can be installed immediately.
1 of these updates is a security update.
To see these additional updates run: apt list --upgradable

The list of available updates is more than a week old.
To check for new updates run: sudo apt update

Last login: Wed Nov 10 05:48:02 2021 from your_ip
WS3C@NIXEASY:~$

WS3C@NIXEASY:~$ cd /home/flag
WS3C@NIXEASY:/home/flag$ ls
flag.txt
WS3C@NIXEASY:/home/flag$ cat flag.txt
||HTB{this_is**********||
```

Great Success!
## 3. Summary
This exercise showed us usage of FTP via non-standard ports, passwordless login via SSH with a keyfile and hidden directories, we will build on these skills in the medium room.
