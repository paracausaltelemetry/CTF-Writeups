---
title: Monday Monitor
summary: TryHackMe endpoint monitoring room using Wazuh and Sysmon logs to trace initial access, scheduled task persistence, user creation, credential dumping, and data exfiltration.
date: 2026-07-02
tags: [TryHackMe, SOC, Wazuh, Sysmon, Endpoint Monitoring, Windows, Incident Response]
difficulty: easy
os: Windows
url: https://tryhackme.com/room/mondaymonitor
---

# CTF Room: Monday Monitor
- [Link to room](https://tryhackme.com/room/mondaymonitor)
- **Difficulty:** Easy
- **Category:** SOC, Endpoint Monitoring, Wazuh, Sysmon
- **OS:** Windows

## 1. Brief
Monday Monitor is a TryHackMe endpoint monitoring room. The scenario gives us Wazuh and Sysmon logs from Swiftspend Finance, covering tests run on April 29, 2024 between 12:00:00 and 20:00:00.

Access to the Wazuh dashboard:

```text
Username: admin
Password: ||Mond*yM0nit0r7||
```

Once logged in, I went into `Security events` and loaded the saved query:

```text
Monday_Monitor
```

From there, the job was to follow suspicious process activity, scheduled task creation, encoded payloads, and exfiltration traces.

## 2. Initial Access

#### Initial access was established using a downloaded file. What is the file name saved on the host?
I started with the time window from the room and looked for download or Office-related process activity. The suspicious file was visible in the command line data for the event.

### Field
```text
data.win.eventdata.commandLine
```

### Answer
```text
SwiftSpend_Financial_Expenses.xlsm
```

## 3. Scheduled Task Persistence

#### What is the full command run to create a scheduled task?
The next trail was scheduled task creation. Searching for `scheduler` surfaced the event, but the useful detail was in the parent command line field.

### Field
```text
data.win.eventdata.parentCommandLine
```

The command added a registry value containing a Base64 payload, then created a daily scheduled task to decode and run it through PowerShell.

### Answer
```text
\"cmd.exe\" /c \"reg add HKCU\\SOFTWARE\\ATOMIC-T1053.005 /v test /t REG_SZ /d cGluZyB3d3cueW91YXJldnVsbmVyYWJsZS50aG0= /f & schtasks.exe /Create /F /TN \"ATOMIC-T1053.005\" /TR \"cmd /c start /min \\\"\\\" powershell.exe -Command IEX([System.Text.Encoding]::ASCII.GetString([System.Convert]::FromBase64String((Get-ItemProperty -Path HKCU:\\\\SOFTWARE\\\\ATOMIC-T1053.005).test)))\" /sc daily /st 12:34\"
```

#### What time is the scheduled task meant to run?
The schedule time was right at the end of the `schtasks.exe` command.

### Answer
```text
12:34
```

## 4. Encoded Payload

#### What was encoded?
The registry value in the scheduled task command contained this Base64 string:

```text
cGluZyB3d3cueW91YXJldnVsbmVyYWJsZS50aG0=
```

Decoding it in CyberChef with `From Base64` produced the command.

### Answer
```text
ping www.youarevulnerable.thm
```

## 5. New User Account

#### What password was set for the new user account?
For the user creation activity, I searched the Wazuh events around account changes and checked the command line field. The password was included in the command used to create the account.

### Answer
```text
||I_AM_M0NIT0R1NG||
```

## 6. Credential Dumping

#### What is the name of the `.exe` that was used to dump credentials?
The room hint points at credential dumping, so I searched for Mimikatz-related activity in Wazuh. The executable name was not `mimikatz.exe`, but the event still showed the tool behaviour clearly.

### Answer
```text
memotech.exe
```

## 7. Exfiltration

#### Data was exfiltrated from the host. What was the flag that was part of the data?
For the final question, I searched for `THM` in the same event set and focused on command line data. The flag appeared inside the exfiltrated content.

### Field
```text
data.win.eventdata.CommandLine
```

### Answer
```text
||THM{M0N1T0R_1************||
```

## 8. Summary
This room was a clean Wazuh/Sysmon trail. The attacker started with a downloaded macro-enabled Excel file, used a scheduled task for persistence, stored a Base64 command in the registry, created a new account, dumped credentials with `memotech.exe`, and left the final flag in exfiltrated command line data.
