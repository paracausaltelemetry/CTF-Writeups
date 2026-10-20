---
title: Snapped Phish-ing Line
summary: TryHackMe phishing investigation room analysing malicious emails, redirection URLs, an exposed phishing kit, captured credentials, VirusTotal results, and a decoded flag.
date: 2026-07-02
tags: [TryHackMe, Phishing, Email Analysis, Phishing Kit, VirusTotal, CyberChef, IOCs]
difficulty: easy
os: Linux
url: https://tryhackme.com/room/snappedphishingline
---

# CTF Room: Snapped Phish-ing Line
- [Link to room](https://tryhackme.com/room/snappedphishingline)
- **Difficulty:** Easy
- **Category:** Phishing Analysis, Email Investigation, Threat Intelligence
- **OS:** Linux

## 1. Brief
Snapped Phish-ing Line is a TryHackMe phishing investigation room. Multiple SwiftSpend Financial employees received suspicious emails, and some users had already entered credentials into the phishing page.

The investigation covered:

- Email artefacts and sender details
- Attachment URL analysis
- Phishing page impersonation
- Exposed phishing kit files
- VirusTotal enrichment
- Captured credential logs
- Phishing kit source review

## 2. Email Review

#### Which individual received the email regarding a Quote for Services Rendered?
I started by reviewing the email samples in the `phish-emails` folder. The email about a quote for services rendered was addressed to William McClean.

### Answer
```text
William McClean
```

#### What email address was used by the adversary to send the phishing emails?
I checked the email source rather than relying only on the mail client display view.

### Answer
```text
||Accounts.Payable@groupmarketingonline.icu||
```

## 3. Attachment And Redirection

#### What is the root domain of the redirection URL found within the file?
The attachment in the email addressed to Zoe Duncan contained a redirection URL. The root domain was:

### Answer
```text
kennaroads.buzz
```

#### Which company is the login page impersonating?
Opening the attachment in the lab VM browser led to a fake login page impersonating Microsoft.

### Answer
```text
microsoft
```

## 4. Exposed Phishing Kit

#### What is the name of the archive file?
The attacker left files exposed under the `/data` directory.

### Path
```text
hxxp://kennaroads[.]buzz/data/
```

The phishing kit archive was visible there.

### Answer
```text
update365.zip
```

#### Using the `sha256sum` command, what is the SHA256 hash of the file?
I downloaded the phishing kit archive inside the VM and hashed it.

### Command
```bash
sha256sum update365.zip
```

### Answer
```text
||ba3c15267393419eb08c7b2652b8b6b39b406ef300ae8a18fee4d16b19ac9686||
```

#### Aside from phishing, what other threat category is assigned to the ZIP archive?
VirusTotal also categorised the archive as a trojan.

### Answer
```text
trojan
```

#### How many files are contained within the archive?
The VirusTotal details page showed the archive contents count.

### Answer
```text
49
```

## 5. Captured Credentials

#### What is the email address of the user who submitted their credentials more than once?
The exposed phishing site also had a credential log under the `Update365` path.

### Path
```text
hxxp://kennaroads[.]buzz/data/Update365/
```

Reviewing the log showed one user had submitted credentials more than once.

### Answer
```text
||michael.ascot@swiftspend.finance||
```

#### What email address is used by the adversary to collect compromised credentials?
After extracting the phishing kit, I reviewed `submit.php`. The collection email was hardcoded in the script.

### File
```text
submit.php
```

### Answer
```text
||m3npat@yandex.com||
```

## 6. Flag Decoding

#### Using CyberChef to decode the flag, what is the secret value?
The phishing URL had a `flag.txt` file exposed.

### Path
```text
hxxp://kennaroads[.]buzz/data/Update365/office365/flag.txt
```

The file contained:

```text
fUxSVV8zSHRfaFQxd195NExwe01IVAo=
```

In CyberChef, I used:

```text
From Base64
Reverse
```

### Answer
```text
||THM{pL4y_w1***********||
```

## 7. Summary
The emails led to a malicious attachment and redirection chain hosted under `kennaroads.buzz`. The fake login page impersonated Microsoft, and the attacker accidentally exposed both the phishing kit archive and credential logs.

Hashing and checking the kit in VirusTotal added enrichment, while `submit.php` revealed where stolen credentials were sent. The final exposed `flag.txt` value needed Base64 decoding and reversing to recover the secret.
