---
title: What the Shard Displayed
summary: Reconstruct a sigrok/PulseView logic-analyzer capture of an embedded display device to recover the image it painted, and the token drawn with it.
date: 2026-07-31
tags: [Hack The Box, Cyber Apocalypse 2026, Hardware]
difficulty: medium
os: N/A
---

# What the Shard Displayed (Medium Hardware) - Writeup

## Challenge

> The Signet shattered, the great houses fell to arguing with steel, and Alyss,
> Queen of Quiet Marches, began sending her dead to watch the living. One of
> her crows fell over our winter line, a maker's device bound beneath its
> wing, an eye she threaded through dead flesh to count our banners. Fed a
> current, it still wakes: it checks its roost, marks the hour it last saw us,
> and paints what it saw onto its pane. Reconstruct it, and learn what her eye
> found of us before the Hollow Host moves.

Provided: `capture.sr`, a sigrok/PulseView logic analyzer capture.

## Flag

```
||HTB{3v3ry_crow_***************||
```

## Step 1: unpack the capture

`.sr` files are just zip archives:

```bash
unzip capture.sr -d sr_extract
cat sr_extract/metadata
```

```ini
[global]
sigrok version=0.5.2

[device 1]
capturefile=logic-1
total probes=8
samplerate=2 MHz
total analog=0
probe1=D0
probe2=D1
unitsize=1
```

Eight digital channels captured at 2 MHz, but only D0 and D1 are named. Sample
data is split across 196 `logic-1-N` chunk files (`unitsize=1`, one byte per
sample, all 8 channels packed in) that concatenate in numeric order to 4,000,000
total samples, i.e. exactly 2 seconds of capture.

## Step 2: identify the bus

```python
data = b"".join(open(f,"rb").read() for f in sorted_chunks)
d0 = [(b>>0)&1 for b in data]
d1 = [(b>>1)&1 for b in data]
```

Only four distinct raw byte values appear across the whole capture
(`0xf0`-`0xf3`), meaning channels D2-D7 never move, so the interesting signal
lives entirely in bits 0-1. Both lines idle high, and the real traffic doesn't
start until sample ~1,962,470 (about 0.98s in), everything before that is idle.

Counting transitions in the active region:

```
D0 transitions: 64642
D1 transitions: 4256
```

D0 toggles far more often than D1, the signature of a clock line versus a data
line. Two wires, both idle-high, one clock and one data: **I2C**, D0 = SCL,
D1 = SDA.

## Step 3: decode I2C by hand

No `sigrok-cli`/`libsigrokdecode` was available in the working environment, so
I2C was decoded directly from the two bit-streams:

- **START**: SDA falls while SCL is high
- **STOP**: SDA rises while SCL is high
- **data bit**: sampled on every SCL rising edge

```python
events = []
prev_scl, prev_sda = scl[0], sda[0]
for i in range(1, n):
    cur_scl, cur_sda = scl[i], sda[i]
    if cur_scl == 1 and prev_scl == 1:
        if prev_sda == 1 and cur_sda == 0: events.append(('START', i))
        elif prev_sda == 0 and cur_sda == 1: events.append(('STOP', i))
    if prev_scl == 0 and cur_scl == 1:
        events.append(('BIT', i, cur_sda))
    prev_scl, prev_sda = cur_scl, cur_sda
```

Result: 200 START/STOP pairs bounding 200 transactions, 32,321 bit edges.
Grouping each transaction's bits into 9-bit frames (8 data bits MSB-first + 1
ACK/NACK bit) and splitting off the leading address byte (7-bit address + R/W)
gives clean, fully-ACKed I2C transactions, no bus errors or corrupted frames
anywhere in the capture.

## Step 4: identify the devices on the bus

Three I2C addresses appear:

| Address | Device | Evidence |
|---|---|---|
| `0x3C` | SSD1306 128x64 OLED | init sequence `AE D5 80 A8 3F D3 00 40 8D 14 20 00 A1 C8 DA 12 81 CF D9 F1 DB 40 A4 A6 AF` is the textbook SSD1306 power-on init, control bytes `0x00` (command) / `0x40` (data) confirm it |
| `0x68` | DS3231 RTC | single register read returning `32 56 00 07 07 02 00` in BCD (seconds, minutes, hours, day, date, month, year) |
| `0x50` | EEPROM / roost store | a 48-byte block read, "it checks its roost" |

This matches the flavor text device-for-device: the "eye" is the OLED, "marks
the hour it last saw us" is the RTC read, and "checks its roost" is the EEPROM
read, all wired to one crow.

## Step 5: reconstruct the display

The 200 transactions split into three full-screen SSD1306 update sequences
(each opens with `Set Column Address 0x21 00 7F` + `Set Page Address 0x22 00
07`, i.e. the whole 128x64 panel), interleaved with the RTC and EEPROM reads:

1. **txns 0-67**: full init + first framebuffer write, followed by the RTC read at txn 66-67
2. **txns 68-134**: second framebuffer write, followed by the EEPROM read at txn 133-134
3. **txns 135-199**: third and final framebuffer write

A small SSD1306 emulator was enough to reconstruct each frame: track the
current column/page pointers from `0x21`/`0x22` commands, then write each
subsequent `0x40`-prefixed data byte into a page/column framebuffer,
auto-wrapping column-to-next-page exactly like the real controller.

```python
if ctrl == 0x00:            # command stream
    ...  # handle 0x21 (col range), 0x22 (page range)
elif ctrl == 0x40:          # data stream
    for byte in payload:
        fb[cur_page][cur_col] = byte
        cur_col += 1
        if cur_col > col_end:
            cur_col = col_start
            cur_page += 1
```

Rendering each of the three segments as a 1-bpp image and upscaling:

**Frame 1** (splash, drawn immediately after init): an eye.

![eye splash](frame0_splash.png)

**Frame 2** (drawn right after the RTC read): the last-seen timestamp.

![05:17](frame1_boot.png)

**Frame 3** (drawn right after the EEPROM/roost read, the final and lasting
state of the display): two lines of text.

![flag](frame2.png)

Read directly off the reconstructed panel:

```
HTB{3v3ry_crow_
w3ars_h3r_3y3s}
```

## Flag

```
||HTB{3v3ry_crow_***************||
```

## Notes

- The EEPROM payload (48 bytes read from `0x50`) never resolves to readable
  text under any obvious transform and isn't needed, once the third frame is
  rendered the flag is sitting directly on the panel in plain text. It's
  narrative dressing for "checks its roost," not a second-stage encode.
- The whole reconstruction only needed two real facts: which two of the eight
  captured channels actually moved, and that toggling frequency alone
  (not signal names) is enough to tell an I2C clock from its data line.
