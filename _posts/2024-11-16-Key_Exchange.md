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
  $puzzle_i \leftarrow E(0^{96} || P_i, "Puzzle \space \# \space x_i" \| k_i)$

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
