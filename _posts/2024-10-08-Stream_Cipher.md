---
title: "Stream Cipher - A Formal Review"
date: 2024-10-08
---

# Stream Cipher
## Symmetric Ciphers
A cipher defined over (K, M, C) is a pair a of "efficient" algorithms (E, D) where
```math
E: K \times M \rightarrow C \newline  
D: K \times C \rightarrow M
```
such that 
```math
D(K, E(K, m)) = m \space\space \forall m \in M, \forall k \in K
```

## Information Theoretic Security
- Secure cipher means cipher text should reveal no "info" about plain text
- A cipher has perfect secrecy if for every two messages m0 and m1, the probability of encrypting m0 with k is equal to the probably of encrypting m1 with k. That means given cipher text, we can't tell if the message is m0 or m1 for all m0, m1.
- In theory, for having perfect secrecy, key len >= msg len

## One Time Pad
- Key as long as the message in theory, impractical
- Secure and good cipher
- OTP has perfect secrecy

## Stream Cipher
- Replace "random" key by "pseudorandom" key
- Pseudo-random generator is a function that maps a smaller seed space to a much larger random output
- Use seed as the key, use generator with the key to generate a random output, then xor with the message

## Negligible and non-negligible
In practice: eps is a scalar and
- eps non-neg: eps >= 1/2^30 (likely to happen over 1GB of data)
- eps negligible: eps <= 1/2^80 (won't happen over life of key)

## WEP
- PRG's seed = $IV || k$
- 24-bit IV -> 16 MB, it will be recycled
- long-term k, never changes
- **Two-time pad**
- **Related key**

## Attack on Stream Cipher
### Attack 1: Two-time pad is insecure
- Never use stream cipher key more than once
```math
c1 \leftarrow m1 \oplus PRG(k)\newline
c2 \leftarrow m2 \oplus PRG(k)\newline
```
Then 
```math
c1 \oplus c2 \rightarrow m1 \oplus m2
```
### Attack 2: No integrity
- xOr cipher text with a value, the plain text will be affected as well when decrypted
```math
c \leftarrow m1 \oplus PRG(k)\newline
m2 \leftarrow c \oplus p\newline
m2 \oplus PRG(k) \rightarrow m1 \oplus p
```
## Real-world Stream Cipher
### RC4
- Bias in initial output: Pr(2nd byte = 0) = 2/256
- Probability of (0, 0) is 1/256^2 + 1/256^3
- Related-key attack

### CSS
- PRG is Linear feedback shift register (LFSR)
- Badly broken

### eStream
- PRG: $\{{0, 1}\}^s \times R \rightarrow \{{0, 1}\}^n$
- R is a nonce. A nonce is a non-repeating value for a given key $E(k, m ; r) = m \oplus PRG(k; r)$
- The pair (k, r) is never used more than once since r is a nonce

### eStream: Salsa 20
- We basically extend key, nonce, and counter to 64 bytes, then apply a function h that transforms 64 bytes to 64 bytes again and again for 10 times.
- Then do addition between the final output and the input
- This whole process above is the function H
```math
Salsa20: \{{0, 1}\}^128 x \{{0, 1}\}^64 \rightarrow \{{0, 1}\}^n\newline
Salsa20(k; r) = H(k, (r, 0)) || H(k, (r, 1)) || ...
```

## PRG Security Definition
- Goal: Define what it means PRG is indistinguishable from a truly random generator, or the output of PRG given a key is indistinguishible from an uniform distribution

### Statistical Tests
- In order to verify that output of PRG is "random", we need statistical tests
- Statistial test is an algorithm A such that A(x) outputs "0" or "1". It means the input you give me looks random or not

### Advantage
- The difference between the probability of a statistical test on a pseudo-random generator output given a key and the probability of the statistical test on a truly random output
```math
Adv_PRG[A, G] = |Pr[A(G(k)) = 1] - Pr[A(r) = 1]| \in [0, 1]
```
- If Adv is close to 1, A can distinguish G from random or A broke G
- Adv is close to 0, A can not distinguish G from random

### Secure PRGs
- A PRG is secure if there exists no efficient statistical test that can distinguish the PRG's output from a truly random generator or the advantage is negligible
- A secure PRG is unpredictable

### Semantic Security
- An encryption scheme is semantically secure if for all efficient algorithm A, the advantage of an adversary over [A, E] is negligible