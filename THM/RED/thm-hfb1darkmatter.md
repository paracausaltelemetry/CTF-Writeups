---
title: DarkMatter
summary: TryHackMe Hackfinity Battle room where a ransomware's 128-bit RSA public key is factored with Fermat's method to recover the private key and decrypt the hostage files.
date: 2026-10-24
tags: [TryHackMe, Cryptography, RSA, Fermat Factorisation, Ransomware]
difficulty: easy
os: N/A
url: https://tryhackme.com/room/hfb1darkmatter
---

# CTF Room: DarkMatter
- [Link to room](https://tryhackme.com/room/hfb1darkmatter)
- **Difficulty:** Easy
- **Category:** Cryptography, RSA

## 1. Brief
Hackfinity High School caught a ransomware variant called DarkInjector. It encrypted some documents and threw up a ransom window demanding a decryption key. The whole room comes down to one mistake the malware author made: the RSA key protecting everything is only 128 bits, which is small enough to factor in seconds. Recover the private key, feed it back into the ransom window, and the files decrypt.

## 2. Finding the Public Key
The ransomware left its RSA public key sitting in `/tmp`.

```text
n = 340282366920938460843936948965011886881
e = 65537
```

The scheme is the usual ransomware pattern: a random AES key encrypts the files, and this RSA key encrypts the AES key. So RSA is the only lock that matters. Break it and everything underneath falls open.

## 3. Why This RSA Is Broken
RSA security rests on `n` being too big to factor. Here `n` is 128 bits. Real RSA uses 2048 bits or more, so 128 bits is not a weak key, it is no key at all.

```bash
python3 -c "print((340282366920938460843936948965011886881).bit_length())"
```
```text
128
```

Normally I would drop `n` into [FactorDB](https://factordb.com/) and let it hand back the factors, but FactorDB was down when I tried. No matter, a key this size is trivial to factor myself.

## 4. Factoring with Fermat's Method
Fermat's method is the right tool when the two primes are close together, which is the common case for a lazily generated key. It writes `n` as a difference of squares, `n = a^2 - b^2 = (a - b)(a + b)`, by starting at `a = ceil(sqrt(n))` and stepping up until `a^2 - n` is a perfect square. When the primes are near each other, that takes almost no steps.

The author wrote theirs in Go, but the method is a few lines in anything. In Python:

```python
import math

n = 340282366920938460843936948965011886881
e = 65537

a = math.isqrt(n)
if a * a < n:
    a += 1
while True:
    b2 = a * a - n
    b = math.isqrt(b2)
    if b * b == b2:
        break
    a += 1

p, q = a - b, a + b
phi = (p - 1) * (q - 1)
d = pow(e, -1, phi)
print("p =", p)
print("q =", q)
print("d =", d)
```
```text
p = 18446744073709551533
q = 18446744073709551557
d = 196442361873243903843228745541797845217
```

The two primes differ by only 24, so Fermat lands on them almost immediately. With `p` and `q` I compute phi(n) = (p-1)(q-1) and then the private exponent `d` as the modular inverse of `e` mod phi(n). That is the whole private key.

The private exponent is:

```text
d = ||196442361873243903843228745541797845217||
```

> **Tip:** If you would rather not write code, [dcode.fr/rsa-cipher](https://www.dcode.fr/rsa-cipher) will take `n`, `e` and the ciphertext and do the factoring and decryption in the browser.

## 5. Decrypting the Files
Back in the ransomware window, I entered `d` as the decryption key. It accepted it and decrypted the two `.docx` files on the Desktop. The grade sheet held the flag.

## 6. Flag / Proof

| Flag | Value |
|:--|:--|
| Flag | `||<paste first half, star the rest>||` |

## 7. Summary
DarkMatter is a one-trick room, and the trick is recognising a toy-sized RSA key.

- **128-bit `n`.** The public key in `/tmp` is far too small. RSA needs 2048+ bits; 128 bits factors in seconds.
- **Fermat because the primes are close.** `p` and `q` differ by 24, so Fermat's difference-of-squares method finds them almost instantly, no FactorDB needed.
- **`d` from the primes.** phi(n) = (p-1)(q-1), then `d` is the inverse of `e` mod phi(n). Entering `d` in the ransom window decrypts the files and reveals the flag.

**Tools used:** `python3` (Fermat factorisation and modular inverse), optionally `dcode.fr` or FactorDB.
