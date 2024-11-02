---
title: "Message Integrity"
date: 2024-11-01
---
## Definition

A MAC is defined as a pair of signing and verifying algorithms over $(K, M, T)$:

- $S(k, m)$ outputs a tag in T
- $V(k, m, t)$ outputs yes or no

![MAC Definition](https://raw.githubusercontent.com/da0p/GithubPage/main/docs/assets/MAC_Definition.drawio.png)

## Secure MACs

- Attacker's power: chosen message attack. For $m_1, m_2, ..., m_q$, attacker is given $t_i \leftarrow S(k, m_i)$
- Attacker's goal: existential forgery. If the attacker can produce some new valid message/tag pair (m, t) and (m, t) $\not\in$ $\{(m_1, t_1), ..., (m_q, t_q)\}$
- If the attacker cannot produce a valid tag for a new message

- Any secure PRF directly gives a secure MAC. For a PRF $F: K \times X \rightarrow Y$ define a MAC $I_F = (S, V)$ as: $S(k, m) = F(k, m)$ and $V(k, m, t)$ output yes if $t = F(k, m)$ and no otherwise

- If Y is large, it's a secure MAC

- Truncation of a secure MAC to w bits is still a secure MAC if w is still large enough (at least 80 bits)

## Encrypted CBC MAC

Let $F: K \times X \rightarrow X$ be a PRP. We can define a new PRF as $F_ECBC \times X^{\leq L} \rightarrow X$

In CBC MAC, we need at least two keys for security

![ECBC](https://raw.githubusercontent.com/da0p/GithubPage/main/docs/assets/raw_cbc.drawio.png)

ECBC-MAC is commonly used as an AES-baed MAC

- CCM encryption mode (used in 802.11i)
- NIST standard called CMAC

Padding must be added even if the message is a multiple of a block. And for security, padding must be invertible. ISO standard states that padding with "1000...". The "1" indicates the beginning of the pad

## NMAC (nested MAC)

Let $F: K \times X \rightarrow K$ be a PRF. We can define a new PRF $F_{NMAC}: K^2 \times X^{\leq L} \rightarrow K$

![NMAC](https://raw.githubusercontent.com/da0p/GithubPage/main/docs/assets/nmac.drawio.png)

NMAC is not usually used with AES or 3DES since the need to change AES key on every block requires re-computing AES key expansion. However, NMAC is the basis for a popular MAC called HMAC

## CMAC (NIST standard)

Variant of CBC-MAC where $key = (k, k_1, k_2)$

- No final encryption step (extension attack is avoided by the last key xor)
- No dummy block needed (ambiguity is resolved by the use of $k_1$ or $k_2$)

![CMAC](https://raw.githubusercontent.com/da0p/GithubPage/main/docs/assets/cmac.drawio.png)

## Security Bounds

After signing $|X|^{1/2}$ messages with ECBC-MAC or $|K|^{1/2}$ messages with NMAC, the MACs become insecure