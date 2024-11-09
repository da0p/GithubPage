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