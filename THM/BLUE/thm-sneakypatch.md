---
title: Sneaky Patch
summary: TryHackMe Linux forensics room investigating a suspicious kernel module and recovering a hidden flag from a backdoored `.ko` file.
date: 2026-07-01
tags: [TryHackMe, Linux Forensics, Kernel, Rootkit, Module Analysis, CyberChef]
difficulty: easy
os: Linux
url: https://tryhackme.com/room/hfb1sneakypatch
---

# CTF Room: Sneaky Patch
- [Link to room](https://tryhackme.com/room/hfb1sneakypatch)
- **Difficulty:** Easy
- **Category:** Linux Forensics, Kernel Module Analysis
- **OS:** Linux

## 1. Brief
Sneaky Patch is a TryHackMe live-system forensics room. The scenario says analysts detected suspicious kernel activity, but normal tooling did not reveal the intruder.

The goal was to investigate deeper than the usual userland checks and recover the hidden flag.

## 2. Initial Checks
I started with the normal live-response checks:

```bash
ps aux
crontab -l
ls -la /etc/cron.*
history
```

Nothing obvious stood out. Since the prompt specifically mentioned suspicious kernel activity and deep persistence, the next step was to inspect loaded kernel modules.

## 3. Loaded Kernel Modules
I listed loaded modules and noticed an unusual module named `spatch`.

### Command
```bash
lsmod
```

The name did not look like a standard module for the system, so I checked its metadata.

### Command
```bash
/sbin/modinfo spatch
```

### Output
```text
filename:       /lib/modules/6.8.0-1016-aws/kernel/drivers/misc/spatch.ko
description:    Cipher is always root
author:         Cipher
license:        GPL
srcversion:     81BE8A2753A1D8A9F28E91E
depends:
retpoline:      Y
name:           spatch
vermagic:       6.8.0-1016-aws SMP mod_unload modversions
```

The description and author made it clear this was the suspicious component.

## 4. Inspecting The Module
I switched to root and inspected printable strings inside the kernel object.

### Commands
```bash
sudo su
strings /lib/modules/6.8.0-1016-aws/kernel/drivers/misc/spatch.ko
```

The module contained a direct marker from the attacker.

### Evidence
```text
[CIPHER BACKDOOR] Here's the secret: ||54484d7b73757033725f736e33346b795f643030727d0a||
```

The value after the marker was hex encoded.

## 5. Decoding
I decoded the hex string in CyberChef with `From Hex`.

### Decoded Output
```text
||THM{sup3r_s***********||
```

## 6. Flag

#### What is the flag?
### Answer
```text
||THM{sup3r_s***********||
```

## 7. Summary
The usual process, cron, and history checks did not reveal the compromise. The important clue was the room's focus on kernel activity.

Inspecting loaded modules exposed `spatch`, a suspicious kernel module authored by Cipher. Its metadata and embedded strings revealed a hex-encoded secret, which decoded directly to the flag.
