---
title: Mr. Phisher
summary: TryHackMe phishing analysis room focused on extracting and decoding a flag hidden inside a macro-enabled Word document attachment.
date: 2026-07-01
tags: [TryHackMe, Phishing, Malware Analysis, Macros, VBA, Python]
difficulty: easy
os: Linux
url: https://tryhackme.com/room/mrphisher
---

# CTF Room: Mr. Phisher
- [Link to room](https://tryhackme.com/room/mrphisher)
- **Difficulty:** Easy
- **Category:** Phishing, Macro Analysis, Document Analysis
- **OS:** Linux

## 1. Brief
Mr. Phisher is a short TryHackMe phishing challenge built around a suspicious macro-enabled document attachment.

The document repeatedly asks the user to enable macros. Rather than enabling them, the goal is to inspect the macro code and recover the hidden flag safely.

> **Warning:** Do not enable macros in suspicious documents. Inspect the macro source in a controlled lab environment instead.

## 2. Lab Setup
The provided files were located in the TryHackMe VM at:

```bash
/home/ubuntu/mrphisher
```

I opened the `.docm` file with LibreOffice Writer and inspected the embedded macros.

## 3. Macro Analysis
Inside the document, I found a macro module called `NewMacros`. The useful procedure was named `Format`.

### VBA Macro
```vba
Rem Attribute VBA_ModuleType=VBAModule
Option VBASupport 1
Sub Format()
Dim a()
Dim b As String
a = Array(102, 109, 99, 100, 127, 100, 53, 62, 105, 57, 61, 106, 62, 62, 55, 110, 113, 114, 118, 39, 36, 118, 47, 35, 32, 125, 34, 46, 46, 124, 43, 124, 25, 71, 26, 71, 21, 88)
For i = 0 To UBound(a)
b = b & Chr(a(i) Xor i)
Next
End Sub
```

The macro creates an array of decimal values. It then loops through each value, XORs it with its index, converts the result into a character with `Chr()`, and appends it to the string `b`.

In short, each character is decoded like this:

```text
decoded_character = Chr(array_value XOR index)
```

## 4. Solving With Python
I recreated the VBA logic in Python to decode the flag.

### Decoder
```python
def decode_flag():
    a = [
        102, 109, 99, 100, 127, 100, 53, 62, 105, 57, 61, 106, 62, 62, 55, 110,
        113, 114, 118, 39, 36, 118, 47, 35, 32, 125, 34, 46, 46, 124, 43, 124,
        25, 71, 26, 71, 21, 88
    ]

    flag = ""

    for i in range(len(a)):
        flag += chr(a[i] ^ i)

    return flag


print(decode_flag())
```

### Output
```text
||flag{a39a07a239aacd*******************||
```

## 5. Flag

#### Uncover the flag in the email attachment!
### Answer
```text
||flag{a39a07a239aacd*******************||
```

## 6. Summary
The document did not need to be executed. Inspecting the VBA macro showed that the flag was encoded as decimal values and recovered by XORing each value against its array index.
