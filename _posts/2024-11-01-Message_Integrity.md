---
title: "Message Integrity"
date: 2024-11-01
---
## Definition

A MAC is defined as a pair of signing and verifying algorithms over (K, M, T):

- S(k, m) outputs a tag in T
- V(k, m, t) outputs yes or no

![MAC Definition](https://raw.githubusercontent.com/da0p/GithubPage/main/docs/assets/MAC_Definition.drawio.png)

## Secure MACs

- Attacker's power: chosen message attack. For $m_1, m_2, ..., m_q$, attacker is given $t_i \leftarrow S(k, m_i)$
- Attacker's goal: existential forgery. If the attacker can produce some new valid message/tag pair (m, t) and (m, t) $\not\in$ $\{(m_1, t_1), ..., (m_q, t_q)\}$
- If the attacker cannot produce a valid tag for a new message

- Any secure PRF directly gives a secure MAC. For a PRF $F: K \times X \rightarrow Y$ define a MAC $I_F = (S, V)$ as: $S(k, m) = F(k, m)$ and $V(k, m, t)$ output yes if $t = F(k, m)$ and no otherwise

- If Y is large, it's a secure MAC

- Truncation of a secure MAC to w bits is still a secure MAC if w is still large enough (at least 80 bits)