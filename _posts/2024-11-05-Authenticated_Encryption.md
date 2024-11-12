---
title: "Authenticated Encryption"
date: 2024-11-05
---

## Definition

An authenticated encryption system $(E, D)$ is a cipher where
$$E: K \times M \times N \rightarrow C$$
but
$$D: K \times C \times N \rightarrow M (U)$$

$N$: a nonce
$U$: a special symbol that can say the ciphertext is rejected

**Security**: the system must provide

- Semantic security under a CPA attack, and
- **Ciphertext Integritry**: attacker cannot create new ciphertexts that decrypt properly

Authenticated encryption ensures confidentiality against an active adversary that can decrypt some ciphertexts but does not prevent replay attacks and account for side channels (timing)

## Authenticated Encryption Construction

![Authenticated-Encryption](https://raw.githubusercontent.com/da0p/GithubPage/main/docs/assets/authenticated_encryption.drawio.png)

## Standards

- **GCM**: CTR mode encryption then CW-MAC
- **CCM**: CBC-MAC then CTR mode encryption
- **EAX**: CTR mode encryption then CMAC

All support AEAD or authenticated encryption with associated data. All are nonce-based

## TLS 1.2

![TLS-1.2](https://raw.githubusercontent.com/da0p/GithubPage/main/docs/assets/tls_1_2.drawio.png)

Note that for each key, they include two keys: one for MAC and one for encryption

How TLS record protocol decrypts messages

- Step 1: CBC decrypt record using k_enc
- Step 2: Check pad format: abort if invalid
- Step 3: check tag on [++ctr_b_s || header || data], abort if invalid

The problem in TLS 1.2 is that it differentiates two types of errors and reports two types of errors. There is an attack for it so-called **padding oracle**

### IMAP over TLS

TLS renegotiates key when an invalid record is received. Every five minutes, client sends login message to server with username and password. The **padding oracle** attack can reveal the password in a few hours

## SSH Binary Packet Protocol

![ssh protocol](https://raw.githubusercontent.com/da0p/GithubPage/main/docs/assets/ssh_protocol.drawio.png)

The problem in ssh decryption is that it decrypt the packet length field before even verifying the MAC is correct!

## Deriving Many Keys

Typically, we have a single source key (SK) that is sampled from

- Hardware random number generator
- A key exchange protocol

However, we need many keys to secure session such as:

- unidirectional keys
- Multiple keys for nonce-based CBC

Therefore we need a function so-called KDF (Key Derivation Function) to generate many keys from this one-source key

### Key Derivation Function (KDF)

Let's suppose we have $F$: a PRF with key space K and outputs in $\{0,1\}^n$
and suppose source key SK is uniform in K then

$$KDF(SK, CTX, L) = F(SK, (CTX||0) || F(SK, (CTX||1)) || ... || F(SK, (CTX||L)))$$

$CTX$: a string that uniquely identifies the application

Note that a PRFs are pseudo random only when key is uniform in K. In reality, source key is often not uniformly random:

- Key exchange protocol: key uniform in some subset of K
- Hardware RNG: may produce biased output

### Extract-then-Expand paradigm

- **Step 1**: Extract pseudo-random key k from source key SK. This step will transform a non-uniform SK to an indistinguishable-from-uniform distribution
- **Step 2**: Expand k by using it as a PRF key as before

### HKDF

It works as following:

- **Extract**: Use $k \leftarrow HMAC(salt, SK)$ where $salt$ is the HMAC key and $SK$ is the HMAC data
- **Expand** using HMAC as a PRF with key k

### Password-Based KDF (PBKDF)

If we want to derive keys from passwords, note that

- Do not use HKDF since passwords have insufficient entropy
- Derived keys will be vulnerable to dictionary attacks

PBKDF defenses with **salt** and a **slow hash function**

The standard approach is **PKCS#5** (PBKDF1) in which $H^{(c)}(pwd\mathbin\Vert salt)$ iterating hash function c times

## Deterministic Encryption

Deterministic encryption refers to the encryption of a message that always leads to the same ciphertext. Deterministic encryption is used for instance
in case of a remote database where the user doesn't want to expose the data. It works as long as it encrypts only one message, or let's say in the case
of a remote database, one unique user id corresponds to one encrypted message

### Construction 1: Synthetic IV (SIV)

Let $(E, D)$ be a CPA-secure encryption

$$E(k, m; r) \rightarrow c$$
Let $F: k \times M \rightarrow R$ be a secure PRF

Define $E_{det}((k_1, k_2), m)$ then

- Apply $F(k_1, m) \rightarrow r$
- Apply $E(k_2, m; r) \rightarrow c$
- Output c

Note that this scheme is well-suited for messages longer than one AES block (16 bytes)

SIV ensures ciphertext integrity

![SIV](https://raw.githubusercontent.com/da0p/GithubPage/main/docs/assets/SIV.drawio.png)