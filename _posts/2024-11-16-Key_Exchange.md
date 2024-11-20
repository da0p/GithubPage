---
title: "Key Exchange"
date: 2024-11-16
---

## Merkle Puzzles

It's a very inefficient algorithm to exchange keys. Let's suppose that we have
an encryption scheme $E(k, m)$ as a symmetric cipher with
$k \in \{0, 1\}^{128}$. Then $puzzle(P) = E(P, "message")$ where
$P=0^{96} || b_1 ... b_{32}$. The goal is to find P by trying all 2^{32}
possibilities.

For instance:

- **Alice**: prepare $2^{32}$ puzzles such as for $i=1, ..., 2^{32}$, choose a
  random $P_i \in \{0,1\}^{32}$ and $x_i, k_i \in \{0,1\}^{128}$, set
  $puzzle_i \leftarrow E(0^{96} || P_i, "Puzzle \space$ # $\space x_i" \| k_i)$

- Send $puzzle_i, ..., puzzle_{32}$ to Bob

![Merkle Puzzles](https://raw.githubusercontent.com/da0p/GithubPage/main/docs/assets/merkle_puzzles.drawio.png)

- It's inefficient since Alice takes $O(n)$ to prepare, and Bob needs $O(n)$ to
  choose and solve the puzzle, but an adversary just need only $O(n^2)$ to find
  the key pair

## Diffie-Hellman Protocol

- Fix a large prime $p$ (e.g. 600 digits)

- Fix an integer $g \in \{1, ..., p\}$, then we will have a pair $(p, g)$

- Alice chooses a random $a \in \{1, ..., p - 1\}$

- Bob chooses a random $b \in \{1, ..., p - 1\}$

- Then we have the shared key
  $B^a (mod \space p) = (g^b)^a = k_{AB} = g^{ab} (mod \space p) = (g^a)^b = A^b (mod \space p)$

In comparison with block cipher. The reason is due to there is an known
algorithm (GNFS) to solve this problem

| cipher key size | modulus size |
| --------------- | ------------ |
| 80 bits         | 1024 bits    |
| 128 bits        | 3072 bits    |
| 256 bits (AES)  | 15360 bits   |

Note that Diffie-Hellman protocol is insecure against man-in-the-middle attack
![Diffie-Hellman MITM](https://raw.githubusercontent.com/da0p/GithubPage/main/docs/assets/diffie_hellman_mitm.drawio.png)

## Public Key Encryption

A public-key encryption system is a triple of algorithms $(G, E, D)$

- G: randomized algorithm (key-generating algorithm) outputs a key pair (pk, sk)

- $E(pk, m)$: randomized algorithm that takes $m \in M$ and outputs $c \in C$

- $D(sk, c)$: algorithm that takes $c \in C$ and outputs $m \in M$ or rejection

### Public Key Constructions

Constructions generally rely on hard problems from number theory and algebra

## Number Theory for Public Key Encryption

Some notations:

- N denotes a positive integer

- p denotes a prime

### Greatest Common Divisor

For integers $x, y: gcd(x, y)$ is the greatest common divisor of x, y

Then we have for all integers x y, there exists integers a, b such that
$$a.x + b.y = gcd(x, y)$$

$a, b$ can be found efficiently using the extended Euclid algorithm

if $gcd(x,y)=1$, we say that x and y are relatively prime

### Modular Inversion

Over the rationals, inverse of 2 is 1/2. But in $Z_N$, the inverse of x is an
element y in $Z_N$ such that $x.y = 1$ in $Z_N$. y is denoted as $x^{-1}$

Note that there is a lemma: x in $Z_N$ has an inverse if and only if
$gcd(x, N) = 1$

### Set of Invertible Elements

$Z_N^*$ = (set of invertible elements in $Z_N$) = $\{x \in Z_N: gcd(x, N) = 1\}$

### Fermat's Theorem

Let p be a prime, then in $Z_P$

$$\forall x \in (Z_p)^*: x^{p-1} = 1 $$

We can use Fermat's theorem for generating random primes. Suppose we want to
generate a large random prime, say prime p of length 1024 bits (i.e.
$p \approx 2 ^{1024}$)

Step 1: Choose a random integer $p \in [2^{1024}, 2^{1025} - 1]$

Step 2: Test if $2^{p-1} = 1$ in $Z_p$. If so, output p and stop, if not, go to
step 1

Note that the probability of p is not a prime in this case is negligible or
$$Pr[p \space not \space prime] < 2^{-60}$$

### Euler's Theorem

$(Z_p)^*$ is a **cyclic group**, that is

$$\exists g \in (Z_p)^*$$

such that $\{1, g, g^2, g^3, ..., g^{p-2}\} = (Z_p)^*$

g is called a **generator** of $(Z_p)^*$

Example: p = 7, then
$\{1, 3, 3^2, 3^3, 3^4, 3^5\} = \{1, 3, 2, 6, 4, 5\} = (Z_7)^*$

and remember that not every element is a generator:
$\{1, 2, 2^2, 2^3, 2^4, 2^5\} = \{1, 2, 4\}$

### Order

For $g \in (Z_p)^*$ the set $\{1, g, g^2, g^3, ...\}$, the group generated by g,
denoted $\langle g \rangle$. The order of $g \in (Z_p)^*$ is the size of
$\langle g \rangle$

$$ord_p(g) = \mid \langle g \rangle \mid$$

We have also Lagrange's theorem:

$\forall g \in (Z_p)^*: ord_p(g)$: $ord_p(g)$ divdes $p-1$

### Euler's generalization of Fermat

For an integer N define $\varphi(12) = \|(Z_N)^*\|$

with Euler's $\varphi$ function

Then

$\forall x \in (Z_N)^*$: $x^{\varphi(N)} = 1$ in $Z_N$

Note that generalization of Fermat is the basis of the RSA cryptosystem

### Modular e'th Roots

Easy case: e is relatively prime to p-1

Suppose $gcd(e, p-1) = 1$, then $\forall c \in (Z_p)^*: \exists \space c^{1/e}$
in $Z_p$

Case: $e = 2$

If p is an odd prime, then $gcd(2, p-1) \neq 1$. We have the following
definition:

x in $Z_p$ is a **quadratic residue** if it has a square root in $Z_p$. p is an
odd prime, the number of **quadratic residue** in $Z_p$ is $(p - 1)/2 + 1$

### Euler's Theorem

Given a number x in $(Z_p)^*$, x is a quadratic residue if and only if
$x^{(p-1)/2}=1$ in $Z_p$

Note that: $x \neq 0$, then $x^{(p-1)/2} = (x^{p-1})^(1/2) \in \{1, -1\}$ in
$Z_p$

$x^{(p-1)/2}$ is called the **Legendre Symbol** of x over p

In order to compute square roots mod p

Suppose p = 3 (mod 4), then $c \in Z_p^*$ is a **quadratic residue**, then
$\sqrt{c} = c^{(p+1)/4}$ in $Z_p$

When p = 1 (mod 4), no deterministic algorithm until now, but there exists a
randomized algorithm to compute it efficiently with run time $O(log^3(p))$

### Quadratic Equations mod p

Given $a.x^2 + b.x + c = 0$ in $Z_p$. We have

$x = (-b +- \sqrt{b^2 - 4.a.c}) / 2a$ in $Z_p$

- Find $(2a)^-1$ in $Z_p$ using extended Euclid

- Find square root of $b^2 - 4.a.c$ in $Z_p$ (if one exists) using a square root
  algorithm

### Computing e'th roots mod N

If N is a composite number and e > 1, in order to compute $c^{1/e}$ efficiently
in $Z_N$ if it exists, it's a hard problem and requires factorization of N

### Arithmetic Algorithm

Representing an n-bit integer on a 64-bit machine would be in blocks of 32 bits

- **Modular Addition and subtraction in $Z_p$:** linear time $O(n)$
- **Modular Multiplication in $Z_p$:** naively $O(n^2)$, but with Karatsuba:
  $O(n^{11.585})$
- **Modular Division with remainder in $Z_p$:** $O(n^2)$
- **Modular Exponentiation in $Z_p$**: repeated squaring algorithm
  $\leq 0((logx).n^2) \leq O(n^3)$

### Intractable Problems with Primes

Fix a prime p > 2 and g in $(Z_p)^*$ of order q. Consider the function
$x \rightarrow g^x$ in $Z_p$, then the inverse function $Dlog_g(g^x) = x$ where
x in $\{0, ..., q-2\}$ is a hard problem.

**DLOG**: Let **G** be a finite cyclic group and **g** a generator of G with
$G = \{1, g, g^2, g^3, ..., g^{q-1}\}$ (q is called the order of G). Then, we
say that **DLOG is hard in G** if for all efficient algorithm A:
$$Pr_{g \leftarrow G, x \leftarrow Z_q}[A(G, q, g, g^x) = x] < negligible$$

For instance, we have $(Z_p)^*$ for large p, or elliptic curve groups mod p