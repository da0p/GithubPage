---
title: "Block Cipher"
date: 2024-10-08
---

# Block Cipher

## How it works

![BlockCipher](https://raw.githubusercontent.com/da0p/GithubPage/main/docs/assets/BlockCiphers.png)

## Performance

Block cipher is considerably slower than stream cipher

## PRPs and PRFs

### PRF

Pseudo Random Function (PRF) defined over (K, X, Y):

$$F: K \times X \rightarrow Y \newline$$

K: Key

X: Input

Y: Output

such that exists "efficient" algorithm to evaluate F(k, x)

### PRP

Pseudo Random Permutation (PRP) defined over (K, X):

$$E: K \times X \rightarrow X$$

such that

- Exists "efficient" deterministic algorithm to evaluate E(k, x)
- The function E(k, .) is one-to-one
- Exists "efficient" inversion algorithm D(k, y)

#### Note

- PRP and Block Cipher are exchangable terms
- PRP is also a PRF where X=Y (the input space and output space are the same)
  and is efficiently invertible

### Secure PRFs

Let F: $K \times X \rightarrow Y$ be a PRF $Funcs[X, Y]$: the set of all
functions from X to Y

$S_F$ = $F(k, .)$ such that $k \in K$ and $S_F \in Funcs[X, Y]$

The meaning is that since the set of truly random PRFs is gigiantic, we need a
smaller set. Therefore we define $S_F$ as a pseudo random function or PRF.

A PRF is secure if a random function in $Funcs[X, Y]$ is indistinguishable from
a random function in $S_F$

### PRG (Pseudo-random generator)

Let $F: K \times \{0, 1\}^n \rightarrow \{0, 1\}^n$ be a secure PRF. Then the
following $G: K \rightarrow \{0, 1\}^{nt}$ is a secure PRG:
$$G(k) = F(k, 0) || F(k, 1) || ... || F(k, t)$$

## Feistel Network

![Feistel Network](https://raw.githubusercontent.com/da0p/GithubPage/main/docs/assets/FeistelNetwork.drawio.png)

## DES

DES is a 16-round Feistel network

## AES

AES is a Substitution-Permutation network

![AES](https://raw.githubusercontent.com/da0p/GithubPage/main/docs/assets/AES.drawio.png)

## Incorrect use of a PRP

- Electronic Code Book (ECB) is not semantically secure
  ![ECB](https://raw.githubusercontent.com/da0p/GithubPage/main/docs/assets/ECB.drawio.png)

## Semantic Security for Many-Time Key

- If given a PT message, the CT is always the same, then it's not secure under
  chosen plain-text attack
- If a secret key is to be used multiple times, the CT must be different every
  time

### Solution 1: Randomized Encryption

E(k, m) is a randomized algorithm
![Many-Time Key](https://raw.githubusercontent.com/da0p/GithubPage/main/docs/assets/Many-Time-Key.drawio.png)

- Encrypting same message twice gives different ciphertexts
- Ciphertext must be longer than plaintext or
  $$CT_{size} = PT_{size} + randombit_{size}$$

### Solution 2: Nonce-based Encryption

- **nonce n**: A value that changes from message to message. The key point is
  that (k, n) pair is never used more than once

![Nonce-based-Encryption](https://raw.githubusercontent.com/da0p/GithubPage/main/docs/assets/Nonce-based-Encryption.drawio.png)

- method 1: nonce is a **counter**
  - Used then encryptor keeps state from message to message
  - If decryptor has same state, does not need send nonce with cipher text
- method 2: encryptor chooses a **random nonce**

### Modes of Operation: Many-Time Key (CBC)

#### Construction 1: CBC with random IV

Let (E, D) be a PRP

$E_{CBC}(k, m)$: choose random IV $in$ X and do:

![CBC-IV-Encryption](https://raw.githubusercontent.com/da0p/GithubPage/main/docs/assets/CBC-Random-IV.drawio.png)

- Note that if CBC where attacker can predict the IV, then it is not CPA-secure

#### Construction 2: Nonce-based CBC

- Cipher block chaining with unique nonce: key = (k, k1)
- Unique nonce means: (key, n) pair is used for only one message

### Note: Padding in CBC

- If the last message is shorter than the the block length, a pad is needed.
  Let's suppose we need to pad 5 bytes, then we put 5 bytes with value 5. The
  decryptor will look at the last byte and remove
- If no pad is needed, still need a dummy block 16 bytes with value 16. The
  decryptor will remove that dummy block

#### Construction 2: Random Counter-Mode and Nonce Counter-Mode

- Counter mode can be computed in parallel, not like CBC mode
  ![CBC-IV-Encryption](https://raw.githubusercontent.com/da0p/GithubPage/main/docs/assets/Random-CTR.drawio.png)
