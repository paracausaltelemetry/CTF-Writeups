---
title: Reversing ELF
summary: TryHackMe beginner reverse-engineering room solving a set of ELF crackmes with execution, strings, Ghidra, Cutter, base64 decoding, XOR, and simple argument checks.
date: 2026-07-03
tags: [TryHackMe, Reverse Engineering, ELF, Ghidra, Cutter, Binary Analysis, Purple Team]
difficulty: easy
os: Linux
url: https://tryhackme.com/room/reverselfiles
---

# CTF Room: Reversing ELF
- [Link to room](https://tryhackme.com/room/reverselfiles)
- **Difficulty:** Easy
- **Category:** Reverse Engineering, ELF, Binary Analysis, Purple Team
- **OS:** Linux

## 1. Brief
Reversing ELF is a beginner-friendly reverse-engineering room built around eight small Linux crackmes.

Some binaries give up the answer when you run them. Others need `strings`, Ghidra, Cutter, a debugger, or a tiny Python decode script. The pattern is consistent: find how the binary checks input, recover the expected value, then submit that value as the flag or password.

Tools used:

- `chmod`
- `strings`
- Ghidra
- Cutter
- CyberChef
- Python

## 2. Crackme1
The first binary is a warmup. I made it executable and ran it.

```bash
chmod +x crackme1
./crackme1
```

The binary prints the flag directly:

```text
||flag{not_that*************||
```

The attached walkthrough notes that the binary builds the output by taking an array and adding `0x41`, which is ASCII `A`, to each value. Running it is enough for the room answer, but the decode routine is useful context for later crackmes.

## 3. Crackme2
This one expects an argument. Running `strings` or checking the comparison in Ghidra exposes the password.

The relevant comparison is a straight `strcmp`:

```asm
push    offset s2       ; "||super_secret_password||"
push    eax             ; s1
call    _strcmp
```

That gives the required argument:

```text
||super_secret_password||
```

Passing that value to the binary prints the flag:

```text
||flag{if_i_submit_this_fla*************************||
```

## 4. Crackme3
Crackme3 also wants an argument. In Ghidra, the interesting part is the encoding routine and the string it compares against.

The character set in the function gives away the encoding:

```text
ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/
```

That is base64. The binary encodes our input, checks the encoded length, and compares it with this value:

```text
ZjByX3kwdXJfNWVjMG5kX2xlNTVvbl91bmJhc2U2NF80bGxfN2gzXzdoMW5nNQ==
```

Decoding it gives the answer:

```text
||f0r_y0ur_5ec0nd_le55on_unbase64_4ll_7h3_7h1ng5||
```

CyberChef with `From Base64` handles this.

## 5. Crackme4
Crackme4 performs a small decode before comparing the user input.

In Ghidra, `main` passes a local buffer into `get_pwd`. That function XORs each byte with `0x24`.

The encoded bytes are:

```text
49 5d 7b 49 14 56 17 7b 57 41 47 51 56 17 7b 54 53 40
```

You can solve this by debugging in Cutter and breaking after `get_pwd`, or by applying the XOR yourself. The decoded password is:

```text
||my_m0r3_secur3_pwd||
```

## 6. Crackme5
This one is easy to overthink in Ghidra because the stack variables look split up. Read them together as one string.

The binary stores a sequence of bytes across locals, then compares the decoded value with our input. Decoding those bytes gives:

```text
||OfdlDSA|3tXb32~X3tX@sX`4tXtz||
```

That is the value to submit.

## 7. Crackme6
Crackme6 is more direct. The useful function names are already doing us a favour:

- `compare_pwd`
- `my_secure_test`

`compare_pwd` passes the input into `my_secure_test`, and that function checks the password character by character.

The recovered password is:

```text
||1337_pwd||
```

## 8. Crackme7
Crackme7 starts as a menu program:

```text
[1] Say hello
[2] Add numbers
[3] Quit
```

The normal menu values are `1`, `2`, and `3`, but Ghidra shows another comparison against `0x7a69`.

Converted to decimal, that value is:

```text
31337
```

Entering `31337` calls `giveFlag`.

The flag decode logic uses the same basic idea as Crackme1: start with `A` and add each encoded byte to it.

```python
encoded = [
    0x25, 0x2B, 0x20, 0x26, 0x3A, 0x2C, 0x34, 0x22, 0x27,
    0x1E, 0x31, 0x24, 0x35, 0x24, 0x31, 0x32, 0x28, 0x2D,
    0x26, 0x1E, 0x35, 0x24, 0x31, 0x38, 0x1E, 0x28, 0x23,
    0x20, 0x1E, 0x36, 0x2E, 0x36, 0x3C
]

print("".join(chr(value + 0x41) for value in encoded))
```

That produces:

```text
||flag{much_reversi****************||
```

## 9. Crackme8
Crackme8 checks the command-line argument against a signed integer. Ghidra shows the key comparison value as:

```text
-0x35010ff3
```

Passing the hex-looking value does not work. Treat it as a signed decimal integer instead:

```text
||-889262067||
```

That grants access and calls `giveFlag`.

The flag decode routine again builds a buffer from `A` plus each encoded byte. I used a small Python script to confirm it from the array in Ghidra.

```python
encoded = [
    0x25, 0x2B, 0x20, 0x26, 0x3A, 0x20, 0x33, 0x1E, 0x2B,
    0x24, 0x20, 0x32, 0x33, 0x1E, 0x33, 0x27, 0x28, 0x32,
    0x1E, 0x22, 0x20, 0x25, 0x24, 0x1E, 0x36, 0x2E, 0x2D,
    0x33, 0x1E, 0x2B, 0x24, 0x20, 0x2A, 0x1E, 0x38, 0x2E,
    0x34, 0x31, 0x1E, 0x22, 0x31, 0x24, 0x23, 0x28, 0x33,
    0x1E, 0x22, 0x20, 0x31, 0x23, 0x1E, 0x2D, 0x34, 0x2C,
    0x21, 0x24, 0x31, 0x32, 0x3C
]

print("".join(chr(value + 0x41) for value in encoded))
```

The final flag is:

```text
||flag{at_least_this_cafe_wont_l*****************************||
```

## 10. Summary
This room is a solid first pass through ELF reversing. The early tasks reward basic execution and `strings`; the later ones push you into decompilation, debugger checks, base64, XOR, signed integer conversion, and small decode scripts.

The main lesson is to keep the workflow simple: run the binary, inspect the strings, read the comparison in Ghidra, and only script the parts that repeat.
