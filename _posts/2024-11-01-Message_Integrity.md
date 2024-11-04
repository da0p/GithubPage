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

Let $F: K \times X \rightarrow X$ be a PRP. We can define a new PRF as $F_{ECBC} \times X^{\leq L} \rightarrow X$

In CBC MAC, we need at least two keys for security

![ECBC](https://raw.githubusercontent.com/da0p/GithubPage/main/docs/assets/raw_cbc.drawio.png)

ECBC-MAC is commonly used as an AES-baed MAC

- CCM encryption mode (used in 802.11i)
- NIST standard called CMAC

Padding must be added even if the message is a multiple of a block. And for security, padding must be invertible. ISO standard states that padding with "1000...". The "1" indicates the beginning of the pad

After signing $\|X\|^{1/2}$ messages with ECBC-MAC or $\|K\|^{1/2}$ messages with NMAC, the MACs become insecure

## NMAC (nested MAC)

Let $F: K \times X \rightarrow K$ be a PRF. We can define a new PRF $F_{NMAC}: K^2 \times X^{\leq L} \rightarrow K$

![NMAC](https://raw.githubusercontent.com/da0p/GithubPage/main/docs/assets/nmac.drawio.png)

NMAC is not usually used with AES or 3DES since the need to change AES key on every block requires re-computing AES key expansion. However, NMAC is the basis for a popular MAC called HMAC

## CMAC (NIST standard)

Variant of CBC-MAC where $key = (k, k_1, k_2)$

- No final encryption step (extension attack is avoided by the last key xor)
- No dummy block needed (ambiguity is resolved by the use of $k_1$ or $k_2$)

![CMAC](https://raw.githubusercontent.com/da0p/GithubPage/main/docs/assets/cmac.drawio.png)

## PMAC (Parallel MAC)

Let $F: K \times X \rightarrow X$ be a PRF. Define new PRF such that $F_{PMAC}: K^2 \times X^{\leq L} \rightarrow X$

![PMAC](https://raw.githubusercontent.com/da0p/GithubPage/main/docs/assets/pmac.drawio.png)

Note that $P(k, i)$ is an easy to compute function

One interesting property of PMAC is that PMAC can be quickly updated when one block changes

## One-time MAC and Many-time MAC
Let $(S,V)$ be a secure one-time MAC over (K, M, $\{0,1\}^n$). Let $F:K_F \times \{0,1\}^n \rightarrow \{0,1\}^n$ be a secure PRF.

**Carter-Wegman MAC**: an algorithm that uses one-time MAC to build a many-time MAC. The idea is

$CW((k_1, k_2), m) = (r, F(k_1, r) \oplus S(k_2, m))$ for random $r \leftarrow \{0,1\}^n$

Note that $S(k_2, m)$ is fast, while $F(k_1, r)$ is slow, but only for the last block

## Collision Resistance

Let $H: M \rightarrow T$ be a hash function where $\|M\| >> \|T\|$

A collision for H is a pair $m_0, m_1 \in M$ such that $H(m_0) = H(m_1)$ and $m_0 \neq m_1$

A function H is collision resistant if for all (explicit) efficient algorithms A such that $Adv_{CR}[A,H]$ = $Pr$[Aoutputs collision for H] is negligible

![Collision Resistance](https://raw.githubusercontent.com/da0p/GithubPage/main/docs/assets/collision_resistance.drawio.png)

From this property, we can define a MAC as such

Let $I= (S,V)$ be a MAC for short messages over $(K,M,T)$. Let $H: M^{big} \rightarrow M$.

We can define $I^{big} = (S^{big}, V^{big})$ over $(K, M^{big}, T)$ as:

$$S^{big}(k,m) = S(k, H(m))$$
$$V^{big}(k, m, t) = V(k, H(m), t)$$

Then if I is a secure MAC and H is collision resistant, then $I^{big}$ is a secure MAC

From this definition, note that collision resistance is necessary for security

## Merkle-Damgard

In order to build a collision-resistant hash function for long messages from a collision-resistant hash function for short messages,
we can use Markle-Damgard iterated construction

Given $h: T \times X \rightarrow T$ (compression function)

We obtain $H: X^{\leq L} \rightarrow T$ wtih $H_i$ - chaining variables

![Merkle-Damgard](https://raw.githubusercontent.com/da0p/GithubPage/main/docs/assets/merkle_damgard.drawio.png)

Note that if the message is a muliple of the block size, then a dummy PB must be added at the end

## Constructing Compression Functions

Since if a compression function is collision resistant, then we can build a secure MAC. We need to find a way to build a compression function

A compression function can be built from a block cipher

$E: K \times \{0, 1\}^n \rightarrow \{0, 1\}^n$ a block cipher. The *Davies-Meyer* compression function: $h(H, m) = E(m, H) \oplus H$

Suppose E is an ideal cipher (collection of $\|K\|$ random perms). Finding a collision $h(H,m) = h(H', m')$ takes $O(2^{n/2})$ evaluations of $(E, D)$

## SHA-256

- Merkle-Damgard function
- Davies-Meyer compression function
- Block cipher: SHACAL-2

![SHA-256](https://raw.githubusercontent.com/da0p/GithubPage/main/docs/assets/SHA-256.drawio.png)

## HMAC

Similar construcing process as NMAC

HMAC: $S(k, m) = H(k \oplus opad, H(k \oplus ipad || m))$

![HMAC](https://raw.githubusercontent.com/da0p/GithubPage/main/docs/assets/HMAC.drawio.png)

HMAC is assumed to be a secure PRF