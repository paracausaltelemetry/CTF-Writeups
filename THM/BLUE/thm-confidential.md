---
title: Confidential
summary: TryHackMe PDF forensics room focused on uncovering an obscured QR code and decoding it to retrieve the hidden invite flag.
date: 2026-06-30
tags: [TryHackMe, Digital Forensics, PDF, QR Code, Image Analysis]
difficulty: easy
os: Linux
url: https://tryhackme.com/room/confidential
---

# CTF Room: Confidential
- [Link to room](https://tryhackme.com/room/confidential)
- **Difficulty:** Easy
- **Category:** Digital Forensics, PDF, QR Code
- **OS:** Linux

## 1. Brief
Confidential is a short TryHackMe digital forensics room. The challenge provides a PDF containing a QR code, but part of the QR code is covered by another image layer.

The task is to uncover the QR code and scan it to retrieve the flag.

## 2. Lab Setup
The room runs inside the TryHackMe split-view VM. The required file is located at:

```bash
/home/ubuntu/confidential
```

After starting the machine, I opened the split view and navigated to the `confidential` directory.

## 3. Inspecting The PDF
I opened the PDF from the provided directory and confirmed that the QR code was visible but partially covered.

Because the challenge is focused on recovering the QR code rather than exploiting a service, the important step was to extract or save the PDF content into an image format that could be inspected separately.

### Steps
```bash
Open Split View
Open the confidential directory
View the PDF
Save the PDF content as an image
Open the image with a QR code viewer
```

## 4. Decoding The QR Code
Once the PDF page was saved as an image, I used a QR code viewer to scan the recovered QR code.

The QR code decoded directly to the flag.

## 5. Flag

#### Uncover and scan the QR code to retrieve the flag!
### Answer
```bash
||flag{e08e6ce2f077a1*******************||
```

## 6. Summary
This was a simple but useful reminder that visual redaction inside a document does not always remove the underlying information.

In this case, the PDF still contained enough of the QR code to recover it by exporting the page as an image and scanning it with a QR code reader.
