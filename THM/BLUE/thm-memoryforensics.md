---
title: Memory Forensics
summary: TryHackMe memory forensics room using Volatility to identify profiles, dump and crack Windows hashes, recover console activity, find shutdown time, and extract a TrueCrypt passphrase.
date: 2026-07-01
tags: [TryHackMe, Memory Forensics, Volatility, Windows, Hash Cracking, TrueCrypt]
difficulty: easy
os: Windows
url: https://tryhackme.com/room/memoryforensics
---

# CTF Room: Memory Forensics
- [Link to room](https://tryhackme.com/room/memoryforensics)
- **Difficulty:** Easy
- **Category:** Memory Forensics, Volatility, Windows DFIR
- **OS:** Windows

## 1. Brief
Memory Forensics is a TryHackMe room focused on pulling useful evidence from Windows memory dumps with Volatility.

The room provides separate `.vmem` files for each task. The workflow is simple but useful: identify the correct profile, run the right Volatility plugin, and then extract the evidence needed for each question.

## 2. Task 1 - Introduction

#### I have understood the task and can continue to the questions!
No analysis was required here. This task just confirms that the memory dumps are large and that Volatility is the main tool for the room.

## 3. Task 2 - Login

#### What is John's password?
The first step with a memory image is to identify the right Volatility profile.

### Identify Profile
```bash
volatility -f Snapshot6.vmem imageinfo
```

The suggested profile was `Win7SP1x64`, so I used that for the rest of this task.

To recover local Windows hashes from memory, I used `hashdump`.

### Dump Hashes
```bash
volatility -f Snapshot6.vmem --profile=Win7SP1x64 hashdump
```

After saving John's NTLM hash to a file, I cracked it with John the Ripper and `rockyou.txt`.

### Crack Hash
```bash
john --wordlist=/usr/share/wordlists/rockyou.txt --format=NT hash.txt
```

### Answer
```text
||Charmander999||
```

## 4. Task 3 - Analysis

#### When was the machine last shutdown?
This task used a different memory dump, so I checked the profile again.

### Identify Profile
```bash
volatility -f Snapshot19.vmem imageinfo
```

Again, `Win7SP1x64` was the useful profile. Volatility has a plugin specifically for the last shutdown timestamp.

### Shutdown Time
```bash
volatility -f Snapshot19.vmem --profile=Win7SP1x64 shutdowntime
```

### Answer
```text
2020-12-27 22:50:12
```

#### What did John write?
The room mentioned that John had a command prompt open. For that, I used the `console` plugin to recover console command history and output from memory.

### Console History
```bash
volatility -f Snapshot19.vmem --profile=Win7SP1x64 console
```

The recovered console text contained the flag.

### Answer
```text
||THM{You_f********||
```

## 5. Task 4 - TrueCrypt

#### What is the TrueCrypt passphrase?
The final task focused on a suspected encrypted TrueCrypt volume. If the passphrase was still resident in memory, Volatility could recover it with the TrueCrypt plugin.

### TrueCrypt Passphrase
```bash
volatility -f Snapshot14.vmem --profile=Win7SP1x64 truecryptpassphrase
```

The passphrase was present in memory.

### Answer
```text
||forgetmenot||
```

## 6. Summary
This room is a clean Volatility practice set:

- Use `imageinfo` to identify the right profile.
- Use `hashdump` and John the Ripper to recover John's password.
- Use `shutdowntime` to build the timeline.
- Use `console` to recover command prompt activity.
- Use `truecryptpassphrase` to pull an encryption passphrase from memory.

The main lesson is that memory often contains exactly the evidence disk analysis misses: live credentials, command history, timestamps, and encryption secrets.
