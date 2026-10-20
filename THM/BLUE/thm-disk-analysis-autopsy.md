---
title: Disk Analysis & Autopsy
summary: TryHackMe forensic analysis room using Autopsy to investigate a Windows disk image, recover system details, user activity, network artifacts, tools, and flags.
date: 2026-06-29
tags: [TryHackMe, Autopsy, Disk Forensics, DFIR, Windows, Registry, Artifacts]
difficulty: medium
os: Windows
url: https://tryhackme.com/room/autopsy2ze0
---

# CTF Room: Disk Analysis & Autopsy
- [Link to room](https://tryhackme.com/room/autopsy2ze0)
- **Difficulty:** Medium
- **Category:** Digital Forensics, Disk Analysis, Autopsy
- **OS:** Windows

## 1. Brief
Disk Analysis & Autopsy is a TryHackMe room focused on investigating a Windows disk image using Autopsy.

The VM contains an Autopsy case file and the corresponding disk image. After opening the `.aut` case file, the disk image needs to be re-pointed correctly before reviewing the already-ingested artifacts.

## 2. Lab Setup
The room provides an RDP-accessible Windows machine with Autopsy and the case files already present.

### RDP Details
```bash
IP: MACHINE_IP
Username: administrator
Password: ||letmein123!||
```

After connecting over RDP, I opened the Autopsy case and manually reviewed the discovered artifacts.

## 3. Image And System Information

#### What is the MD5 hash of the E01 image?
The image hash was found in Autopsy under the file metadata for the E01 data source.

### Answer
```bash
||3f08c518adb3b5c1359849657a9b2079||
```

#### What is the computer account name?
This was found under the Operating System Information section.

### Answer
```bash
DESKTOP-0R59DJ3
```

#### List all the user accounts. (alphabetical order)
The users were listed under Operating System User Accounts.

### Answer
```bash
H4S4N,joshwa,keshav,sandhya,shreya,sivapriya,srini,suba
```

#### Who was the last user to log into the computer?
I sorted the Operating System User Accounts by the Date Accessed field.

### Answer
```bash
sivapriya
```

## 4. Network Artifacts

#### What was the IP address of the computer?
The IP address was found in the Look@LAN configuration file.

### Artifact
```bash
Vol3\Program Files (x86)\Look@LAN\irunin.ini
```

### Answer
```bash
192.168.130.216
```

#### What was the MAC address of the computer?
The MAC address was found in the same Look@LAN configuration file under the `LANNIC` value, then formatted with hyphens.

### Artifact
```bash
Vol3\Program Files (x86)\Look@LAN\irunin.ini
```

### Answer
```bash
08-00-27-2C-C4-B9
```

#### What is the name of the network card on this computer?
The network card was found by reviewing the Windows registry through Autopsy.

### Artifact
```bash
SOFTWARE\Microsoft\Windows NT\CurrentVersion\NetworkCards
C:\Windows\System32\config
```

### Answer
```bash
Intel(R) PRO/1000 MT Desktop Adapter
```

#### What is the name of the network monitoring tool?
The tool name was identified from the same installed program and configuration path used for the IP and MAC address.

### Answer
```bash
Look@LAN
```

## 5. User Artifacts

#### A user bookmarked a Google Maps location. What are the coordinates of the location?
The bookmarked location was found under Web Bookmarks in Autopsy.

### Answer
```bash
12°52'23.0"N 80°13'25.0"E
```

#### A user has his full name printed on his desktop wallpaper. What is the user's full name?
I checked the images for each user until I found the desktop wallpaper containing the user's full name.

### Answer
```bash
Anto Joshwa
```

#### A user had a file on her desktop. It had a flag but she changed the flag using PowerShell. What was the first flag?
The original flag was recovered from Shreya's PowerShell history.

### Artifact
```bash
Users\shreya\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt
```

### Answer
```bash
||flag{HarleyQu************||
```

#### The same user found an exploit to escalate privileges on the computer. What was the message to the device owner?
The message was found in `exploit.ps1` on Shreya's desktop.

### Answer
```bash
||flag{I-ha*********||
```

## 6. Tools And Malware Artifacts

#### 2 hack tools focused on passwords were found in the system. What are the names of these tools? (alphabetical order)
These were found by checking H4S4N's downloads and the prefetch evidence for the executables.

### Answer
```bash
Lazagne,Mimikatz
```

#### There is a YARA file on the computer. Inspect the file. What is the name of the author?
I searched for `.yar` and found a shortcut pointing to a Mimikatz ZIP. Inspecting the YARA-related artifact revealed the author.

### Answer
```bash
Benjamin DELPY (gentilkiwi)
```

#### One of the users wanted to exploit a domain controller with an MS-NRPC based exploit. What is the filename of the archive that you found?
The archive was found through the recent documents artifacts.

### Answer
```bash
2.2.0 20200918 Zerologon encrypted.zip
```

## 7. Summary
This room was a useful Autopsy practice case because the answers were spread across several common forensic artifact categories.

The system details came from Autopsy's OS information and file metadata, the network details were recovered from Look@LAN configuration files and registry artifacts, and the user activity was reconstructed from bookmarks, images, PowerShell history, desktop files, downloads, prefetch data, and recent documents.
