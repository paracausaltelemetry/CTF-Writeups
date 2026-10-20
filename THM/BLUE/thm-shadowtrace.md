---
title: Shadow Trace
summary: TryHackMe malware triage room analysing a suspicious Windows updater, extracting IOCs, decoding hidden clues, and correlating EDR alert payloads.
date: 2026-07-01
tags: [TryHackMe, Malware Analysis, DFIR, PE Analysis, IOCs, EDR, CyberChef]
difficulty: easy
os: Windows
url: https://tryhackme.com/room/shadowtrace
---

# CTF Room: Shadow Trace
- [Link to room](https://tryhackme.com/room/shadowtrace)
- **Difficulty:** Easy
- **Category:** Malware Analysis, SOC Triage, Alert Analysis
- **OS:** Windows

## 1. Brief
Shadow Trace is a TryHackMe malware triage room. The scenario provides a suspicious file found on a user's machine and a set of EDR-style alerts to correlate.

The binary for analysis was:

```text
C:\Users\DFIRUser\Desktop\windows-update.exe
```

Useful tools were available under:

```text
C:\Users\DFIRUser\DFIR Tools
```

I used PeStudio and PE-Bear for static PE triage, then CyberChef to decode the suspicious strings and alert payloads.

## 2. Task 1 - Scenario

#### Click here to start the challenge
No answer was required for this task.

## 3. Task 2 - File Analysis

#### What is the architecture of the binary file `windows-update.exe`?
PeStudio identified the file as a 64-bit Windows PE.

### Answer
```text
64-bit
```

#### What is the hash (sha-256) of the file `windows-update.exe`?
The SHA256 hash was visible in PeStudio's file metadata.

### Answer
```text
||B2A88DE3E3BCFAE4A4B38FA36E884C586B5CB2C2C283E71FBA59EFDB9EA64BFC||
```

#### Identify the URL within the file to use it as an IOC
In PE-Bear, I searched strings for `http` and found a suspicious update URL.

### Answer
```text
http://tryhatme.com/update/security-update.exe
```

#### With the URL identified, can you spot a domain that can be used as an IOC?
Filtering around the discovered domain exposed a related suspicious subdomain.

### Answer
```text
responses.tryhatme.com
```

#### Input the decoded flag from the suspicious domain
The suspicious URL path contained a Base64-looking value:

```text
tryhatme.com/||****************************************||
```

Decoding that value in CyberChef produced the flag.

### Answer
```text
||THM{you_g0t_som**************||
```

#### What library related to socket communication is loaded by the binary?
The binary imports showed the Winsock library.

### Answer
```text
WS2_32.dll
```

## 4. Task 3 - Alerts Analysis

#### Can you identify the malicious URL from the trigger by the process `powershell.exe`?
The alert payload contained a Base64 string:

```text
aHR0cHM6Ly90cnloYXRtZS5jb20vZGV2L21haW4uZXhl
```

Decoding it revealed the malicious URL.

### Answer
```text
https://tryhatme.com/dev/main.exe
```

#### Can you identify the malicious URL from the alert triggered by `chrome.exe`?
The Chrome alert contained decimal character codes:

```text
104,116,116,112,115,58,47,47,114,101,97,108,108,121,115,101,99,117,114,101,117,112,100,97,116,101,46,116,114,121,104,97,116,109,101,46,99,111,109,47,117,112,100,97,116,101,46,101,120,101
```

Using `From Decimal` in CyberChef decoded the URL.

### Answer
```text
https://reallysecureupdate.tryhatme.com/update.exe
```

#### What's the name of the file saved in the alert triggered by `chrome.exe`?
The saved filename was visible in the Chrome alert command payload.

### Answer
```text
test.txt
```

## 5. Summary
The suspicious updater was a 64-bit Windows PE with clear network IOCs embedded in its strings and imports. Static analysis exposed a fake update URL, a related suspicious domain, and a Winsock dependency.

The alert review added two more malicious URLs: one Base64 encoded in the PowerShell alert, and one decimal encoded in the Chrome alert. The Chrome payload also showed that the output file was saved as `test.txt`.
