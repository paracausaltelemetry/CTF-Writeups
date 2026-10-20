---
title: Summit
summary: TryHackMe purple-team detection engineering challenge using the Pyramid of Pain to detect malware through hashes, IPs, domains, host artifacts, tool behavior, and attacker procedures.
date: 2026-06-29
tags: [TryHackMe, Purple Team, Detection Engineering, Pyramid of Pain, Sigma, Firewall, DNS, Malware]
difficulty: easy
os: Windows
url: https://tryhackme.com/room/summit
---

# CTF Room: Summit
- [Link to room](https://tryhackme.com/room/summit)
- **Difficulty:** Easy
- **Category:** Purple Team, Detection Engineering, Pyramid of Pain
- **OS:** Windows

## 1. Brief
Summit is a TryHackMe challenge built around an iterative purple-team engagement. PicoSecure has hired an external penetration tester to execute malware samples against a simulated internal workstation.

The goal is to configure detections and prevention rules that stop each malware sample. Each stage moves higher up the Pyramid of Pain, forcing the adversary to spend more effort changing their infrastructure, tools, or behavior.

## 2. Lab Setup
After starting the lab machine, the room provides a hosted application URL.

```bash
https://LAB_WEB_URL.p.thmlabs.com
```

The room notes that the application may take a few minutes to fully deploy. If the page returns `Bad Gateway`, wait a few minutes and refresh.

## 3. Detection Progression

### Sample 1 - File Hash
The first malware sample was blocked using a SHA256 hash detection.

Hashes are high-confidence indicators because they identify a specific file. The downside is that the attacker only needs to alter or recompile the file to generate a new hash.

#### What is the first flag you receive after successfully detecting sample1.exe?
### Answer
```bash
||THM{f3cbf08151a11a6******************||
```

## 4. Sample 2 - IP Address
After the attacker recompiled the malware and changed the file hash, I moved up the Pyramid of Pain and blocked the network indicator.

The malware sandbox showed the external IP address the sample attempted to connect to, so I created an egress firewall rule to block that destination.

#### What is the second flag you receive after successfully detecting sample2.exe?
### Answer
```bash
||THM{2ff48a3421a938b******************||
```

## 5. Sample 3 - Domain Name
The attacker moved to a new IP address, so blocking IPs was no longer enough. The next stable indicator was the domain used by the malware.

### DNS Block
```bash
emudyn.bresonicz.info
```

Blocking the domain meant that new IP addresses behind the same domain would still be stopped.

#### What is the third flag you receive after successfully detecting sample3.exe?
### Answer
```bash
||THM{4eca9e2f61a19ec******************||
```

## 6. Sample 4 - Host Artifact
For the fourth sample, hashes, IPs, and domains were no longer useful. The detection needed to focus on host artifacts left behind by the malware.

The malware modified a Windows Defender registry setting, so I created a Sigma-style registry modification rule.

### Registry Modification Rule
```bash
Registry Key:  HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows Defender\Real-Time Protection
Registry Name: DisableRealTimeMonitoring
Value:         1
ATT&CK ID:     Defense Evasion (TA0005)
```

#### What is the fourth flag you receive after successfully detecting sample4.exe?
### Answer
```bash
||THM{c956f455fc076ae******************||
```

## 7. Sample 5 - Tool Behavior
The fifth sample shifted most of the heavy lifting to the attacker's backend server. Instead of blocking static artifacts, I used the outgoing network connection logs to identify abnormal recurring behavior.

### Network Connection Rule
```bash
Remote IP:           Any
Remote Port:         Any
Size (bytes):        97
Frequency (seconds): 1800
ATT&CK ID:           Command and Control (TA0011)
```

This rule focused on the tool's network behavior rather than a single hash, IP, or domain.

#### What is the fifth flag you receive after successfully detecting sample5.exe?
### Answer
```bash
||THM{46b21c4410e47dc******************||
```

## 8. Sample 6 - Techniques and Procedures
For the final sample, the detection moved to the top of the Pyramid of Pain by focusing on the attacker's behavior.

The previous command logs showed that the attacker repeatedly used `cmd.exe` to reference an exfiltration log in the temp directory. I created a process creation rule to detect that command-line behavior.

### Process Creation Rule
```bash
Process Name: cmd.exe
CommandLine:  Contains %temp%\exfiltr8.log
ATT&CK ID:    Collection (TA0009)
```

#### What is the final flag you receive from Sphinx?
### Answer
```bash
||THM{c8951b2ad24bbcb******************||
```

## 9. Summary
This room was a practical demonstration of why the Pyramid of Pain matters for detection engineering.

The first detections targeted low-level indicators such as file hashes, IP addresses, and domains. These were quick to implement but easy for the adversary to change. As the engagement progressed, the detections moved toward host artifacts, tool behavior, and finally the attacker's techniques and procedures.

By the end, the attacker could no longer bypass the detections with simple recompiles, infrastructure changes, or tooling changes. They would need to retrain and substantially alter their operating procedure, which is exactly the point of climbing the Pyramid of Pain.
