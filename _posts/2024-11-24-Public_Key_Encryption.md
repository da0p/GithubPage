---
title: "Public Key Encryption"
date: 2025-01-05
---

A public-key encryption system is a triple of algorithms $(G, E, D)$ such that

- $G()$: randomized algorithm outputs a key pair $(pk, sk)$
- $E(pk, m)$: randomized algorithm that takes $m \in M$
- $D(sk, c)$: deterministic algorithm that takes $c \in C$ and outputs $m \in M$
  or rejects it

That means: $\forall (pk, sk)$ output by G:
$$\forall m \in M: D(sk, E(pk, m)) = m$$

## Security: Eavesdropping

For $b=0,1$, define experiments EXP(0) and EXP(1) as:
![Public Key Eavesdroppping](https://raw.githubusercontent.com/da0p/GithubPage/main/docs/assets/public_key_eavesdropping_sec.drawio.png)

Given equal-length messages $m_0$ and $m_1$ sent to the challenger. Then
$E=(G, E, D)$ is semantic secure if for all efficient A:
$$Adv_{SS}[A, E] = |Pr[EXP(0)=1] - Pr[EXP(1)=1]| < negligible$$

## Security: Active attacks

What if attacker can tamper with ciphertext?
![Active Attacks](https://raw.githubusercontent.com/da0p/GithubPage/main/docs/assets/public_key_active_attacks.drawio.png)

## Chosen Ciphertext Security

![Chosen ciphertext security](https://raw.githubusercontent.com/da0p/GithubPage/main/docs/assets/public_key_chosen_ciphertext_security.drawio.png)

E is CCA secure if for all efficient A:

$$Adv_{CCA}[A,E] = |Pr[EXP(0)=1] - Pr[EXP(1)=1] < negligible$$

## Trapdoor Functions (TDF)

A trapdoor function $X \rightarrow Y$ is a triple of efficient algorithms
$(G, F, F^{-1})$ such that

- $G()$: randomized algorithm outputs a key pair $(pk, sk)$
- $F(pk, .)$: deterministic algorithm that defines a function $X \rightarrow Y$
- $F^{-1}(sk, .)$: defines a function $Y \rightarrow X$ that inverts $F(pk, .)$

More precisely: $\forall(pk, sk)$ output by G

$$\forall x \in X: F^{-1}(sk, F(pk, x)) = x$$

## Secure Trapdoor Functions (TDFs)

$(G, F, F^{-1})$ is secure if $F(pk, .)$ is a "one-way" function if it can be
evaluated byt cannot be inverted without sk

## Public-key Encryption from TDFs

- $(G, F, F^{-1})$: secure TDF $X \rightarrow Y$
- $(E_s, D_s)$: symmetric authenticated encryption defined over $(K, M, C)$
- $H: X \rightarrow K$ a hash function

We construct a pub-key encryption system $(G, E, D)$: Key generation G: same as
G for TDF

### RSA Encryption

$E(pk, m)$:

$x \leftarrow X$, $y \leftarrow F(pk, x)$

$k \leftarrow H(x), c \leftarrow E_s(k, m)$

output $(y, c)$

### Decryption

$D(sk, (y, c))$:

$x \leftarrow F^{-1}(sk, y)$

$k \leftarrow H(x), m \leftarrow D_s(k, c)$

output $m$

## RSA Trapdoor Permutation

$G()$: choose random primes $p, q \approx 1024$ bits. Set $N = pq$. Choose
integers $e, d$ such that $e.d=1$ (mod $\phi(N)$). Then the output
$pk = (N, e), sk = (N, d)$

## RSA Public Key Encryption

$(E_s, D_s)$: symmetric encryption scheme providing authenticated encryption.

$H: Z_N \rightarrow K$ where K is key space of $(E_s, D_s)$

- $G()$: generate RSA params: $pk = (N, e), sk = (N, d)$
- $E(pk, m)$:
  - choose random $x \in Z_N$
  - $y \leftarrow RSA(x) = x^e, k \leftarrow H(x)$
  - output $(y, E_s(k, m))$
- $D(sk, (y, c)): output $D_s(H(RSA^{-1}(y)), c)$$ or $m$

## RSA Encrytion in Practice

In practice, RSA is given with a symmetric key, then pre-processed before input
into an RSA function

![Chosen ciphertext security](https://raw.githubusercontent.com/da0p/GithubPage/main/docs/assets/rsa_enc_in_practice.drawio.png)

### Preprocessing - PKCS1 v1.5

![PKCS1 v1.5](https://raw.githubusercontent.com/da0p/GithubPage/main/docs/assets/pkcs1_v_1_5.drawio.png)

### Preprocessing - PKCS1 v2.0 (OAEP)

New preprocessing function: OAEP

![OAEP](https://raw.githubusercontent.com/da0p/GithubPage/main/docs/assets/oaep.drawio.png)

RSA is a trap-door permutation -> RSA-OAEP is CCA secure when H, G are random
oracles. In practice, use SHA-256 for H and G

### Preprocessing - OAEP+

For every trap-door permutation $F$, $F-OAEP+$ is CCA secure when $H, G, W$ are
random oracles. During decryption validate $W(m, r)$ field

![OAEP+](https://raw.githubusercontent.com/da0p/GithubPage/main/docs/assets/oaep_plus.drawio.png)

### Preprocessing - SAEP+

RSA (e = 3) is a trap-door permutation -> RSA-SAEP+ is CCS secure when $H, W$
are random oracle

![SAEP+](https://raw.githubusercontent.com/da0p/GithubPage/main/docs/assets/saep_plus.drawio.png)

## ElGamal Public Key System

- Fix a finite cyclic group G (e.g $G = (Z_p)^*$) of order n
- Fix a generator g in G (i.e. $G = {1, g, g^2, g^3, ..., g^{n-1}}$)

![ElGamal](https://raw.githubusercontent.com/da0p/GithubPage/main/docs/assets/ElGamal.drawio.png)

## ElGamal System - A Modern View

- $G$: finite cylic group of order n
- $(E_s, D_s)$: symmetric authenticated encryption defined over $(K, M, C)$
- $H: G^2 \rightarrow K$ a hash function

### ElGamal Encryption

$E(pk = (g, h), m)$

$b \leftarrow Z_n, u \leftarrow g^b, v \leftarrow h^b$

$k \leftarrow H(u, v), c \leftarrow E_s(k, m)$

output $(u, c)$

### ElGamal Decryption

$D(sk = a, (u, c))$

$v \leftarrow u^a$

$k \leftarrow H(u, v), m \leftarrow D_s(k, c)$

output $m$

## Computational Diffie-Hellman Assumption

$G$: finite cylic group of order n

Computational DH (CDH) assumption holds in G if: given $g, g^a, g^b$. It's
impossible to compute $g^{ab}$

For all efficient algorithms A:

$$Pr[A(g, g^a, g^b) = g^{ab}] < negligible$$

where $g \leftarrow$ {generators of G},

$a, b \leftarrow Z_n$

## Hash Diffie-Hellman Assumption

$G$: finite cyclic group of order n, $H: G^2 \rightarrow K$ a hash function

Def: Hash-DH (HDH) assumption holds for $(G, H)$ if

$(g, g^a, g^b, H(g^b, g^{ab})) \approx (g, g^a, g^b, R)$

where $g \leftarrow$ {generators of G},

$a, b \leftarrow Z_n, R \leftarrow K$

## Twin ElGamal

Key Generation:

$g \leftarrow {generators of G}$

$a1, a2 \leftarrow Z_n$

output $pk = (g, h_1 = g^{a1}, h_2 = g^{a2})$

$sk = (a1, a2)$

### Twin ElGamal - Encryption

$E(pk = (g, h_1, h_2), m): b \leftarrow Z_n$

$k \leftarrow H(g^b, h_1^b, h_2^b)$

$c \leftarrow E_s(k, m)$

output $(g^b, c)$

### Twin ElGamal - Decryption

$D(sk = (a1, a2), (u, c))$:

$k \leftarrow H(u, u^{a1}, u^{a2})$

$m \leftarrow D_s(k, c)$

output $m$

The advantage of twin ElGamal is that there is no need for HDH assumption, but
only CDH assumption to prove that twin ElGamal is CCA secure
