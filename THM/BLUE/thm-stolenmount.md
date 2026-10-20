---
title: Stolen Mount
summary: TryHackMe packet forensics room analysing NFS traffic, extracting stolen files from a PCAP, cracking an archive password, and recovering a QR-code flag.
date: 2026-07-01
tags: [TryHackMe, Forensics, PCAP, Wireshark, NFS, CyberChef, QR Code]
difficulty: easy
os: Linux
url: https://tryhackme.com/room/hfb1stolenmount
---

# CTF Room: Stolen Mount
- [Link to room](https://tryhackme.com/room/hfb1stolenmount)
- **Difficulty:** Easy
- **Category:** Packet Analysis, Network Forensics, NFS
- **OS:** Linux

## 1. Brief
Stolen Mount is a TryHackMe packet forensics room. The scenario says an intruder accessed an unauthenticated NFS share and stole classified data.

The provided capture file was located on the lab desktop:

```text
~/Desktop/challenge.pcapng
```

The goal was to recover the contents of the stolen data and extract the flag.

## 2. Initial PCAP Triage
I started with the obvious checks: string searching and looking for exported objects or file paths. Nothing immediately exposed the flag.

Next, I reviewed the protocol statistics in Wireshark. The capture was small, with only one main conversation and around 304 packets. The useful traffic was TCP/NFS, so I focused on following the relevant stream.

## 3. NFS Stream Review
Following the NFS stream revealed several useful artefact names:

```text
creds.txt
secrets.png
hidden_stash.zip
```

The stream also exposed an MD5 hash:

```text
||90eb7723a657b6597100aafef171d9f2||
```

I cracked the hash with CrackStation, which returned the password:

```text
||avengers||
```

## 4. File Extraction
To recover the transferred files, I copied the stream as a hexdump and used CyberChef.

### CyberChef Recipe
```text
From Hexdump
Extract Files
```

This recovered two ZIP-like artefacts. One appeared to be corrupted, but the useful archive contained `secrets.png`.

Using the cracked password allowed the archive contents to be recovered.

### Password
```text
||avengers||
```

## 5. QR Code Recovery
The recovered `secrets.png` file contained a QR code. Scanning the QR code revealed the final flag.

## 6. Flag

#### What is the flag?
### Answer
```text
||THM{n0t_s3cur3**************||
```

## 7. Summary
The PCAP showed NFS activity tied to stolen backup files. The useful evidence was in a single TCP/NFS conversation, where file names and an MD5 hash were visible.

After cracking the MD5 hash, extracting files from the stream with CyberChef, and using the recovered password to open the archive, the stolen `secrets.png` QR code revealed the flag.
