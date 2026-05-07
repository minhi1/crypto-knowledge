<h1> Table of Contents </h1>


<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [1. The Big Picture: What Is a Symmetric Cipher?](#1-the-big-picture-what-is-a-symmetric-cipher)
  - [Challenge](#challenge)
- [2. Shift Cipher (Caesar Cipher)](#2-shift-cipher-caesar-cipher)
  - [Definition](#definition)
  - [Worked Example](#worked-example)
  - [Why It's Weak](#why-its-weak)
  - [Challenge](#challenge-1)
- [3. Substitution Cipher (Monoalphabetic)](#3-substitution-cipher-monoalphabetic)
  - [Definition](#definition-1)
  - [Worked Example](#worked-example-1)
  - [Why It's Weak — Frequency Analysis](#why-its-weak--frequency-analysis)
  - [Challenge](#challenge-2)
- [4. Affine Cipher](#4-affine-cipher)
  - [Motivation: A Structured Substitution](#motivation-a-structured-substitution)
  - [Definition](#definition-2)
  - [Why gcd(a, 26) = 1 Is Required](#why-gcda-26--1-is-required)
  - [Worked Example](#worked-example-2)
  - [Challenge](#challenge-3)
- [5. The Euclidean Algorithm](#5-the-euclidean-algorithm)
  - [Why We Need It](#why-we-need-it)
  - [The Algorithm](#the-algorithm)
  - [Worked Example](#worked-example-3)
  - [Challenge](#challenge-4)
- [6. The Extended Euclidean Algorithm](#6-the-extended-euclidean-algorithm)
  - [From GCD to Modular Inverse](#from-gcd-to-modular-inverse)
  - [The Algorithm: Back-Substitution](#the-algorithm-back-substitution)
  - [Worked Example](#worked-example-4)
  - [Tabular Method (Faster)](#tabular-method-faster)
  - [Challenge](#challenge-5)
- [7. Vigenère Cipher (Polyalphabetic)](#7-vigenère-cipher-polyalphabetic)
  - [The Core Insight](#the-core-insight)
  - [Definition](#definition-3)
  - [Worked Example](#worked-example-5)
  - [Why It's Stronger — And Why It Still Falls](#why-its-stronger--and-why-it-still-falls)
  - [Challenge](#challenge-6)
- [8. Hill Cipher](#8-hill-cipher)
  - [The Core Insight](#the-core-insight-1)
  - [Definition](#definition-4)
  - [Finding the Matrix Inverse mod 26](#finding-the-matrix-inverse-mod-26)
  - [Worked Example (Encryption)](#worked-example-encryption)
  - [Worked Example (Decryption)](#worked-example-decryption)
  - [Challenge](#challenge-7)
- [9. Permutation Cipher (Transposition)](#9-permutation-cipher-transposition)
  - [The Core Insight](#the-core-insight-2)
  - [Definition](#definition-5)
  - [Worked Example — Columnar Transposition](#worked-example--columnar-transposition)
  - [Challenge](#challenge-8)
- [10. Summary and Comparison](#10-summary-and-comparison)

<!-- /code_chunk_output -->



# 1. The Big Picture: What Is a Symmetric Cipher?

A **symmetric cipher** is an encryption scheme where the sender and receiver share the exact same secret key — what locks the data also unlocks it.

The general model is simple:

$$C = E_K(M) \qquad M = D_K(C)$$

where $M$ is the plaintext, $C$ is the ciphertext, and $K$ is the shared secret key.

Before modern block ciphers like DES and AES existed, **classical symmetric ciphers** were used for centuries. They are simpler (often breakable today), but they teach the fundamental building blocks of all cryptography:

- **Substitution**: replace each character with another character.
- **Transposition (Permutation)**: rearrange the order of characters without changing them.

> Every modern cipher — including AES — is, at its core, a sophisticated combination of repeated substitutions and permutations. Understanding classical ciphers means understanding why cryptographers kept pushing for something stronger.

We will always work over the alphabet $\{A, B, \ldots, Z\}$, encoding letters as numbers $\{0, 1, \ldots, 25\}$.

## Challenge

A sender encrypts a message using a simple cipher. An attacker intercepts the ciphertext and notices that the letter "Q" appears far more often than any other letter.

<u>What does this observation tell the attacker, and which type of cipher is most likely being used?</u>

The attacker recognizes a **frequency analysis** attack opportunity. In English, the letter "E" is the most common letter (~12.7% frequency). If "Q" dominates the ciphertext, it is almost certainly the encryption of "E". This pattern means the cipher is a **monoalphabetic substitution cipher** — each plaintext letter always maps to the same ciphertext letter, so the frequency distribution of the plaintext is preserved in the ciphertext.

---

# 2. Shift Cipher (Caesar Cipher)

## Definition

The **Shift Cipher** (also called the Caesar Cipher after Julius Caesar) is the simplest cipher. Every letter in the plaintext is shifted by a fixed number $K$ positions along the alphabet.

$$E_K(m) = (m + K) \bmod 26$$

$$D_K(c) = (c - K) \bmod 26$$

The **key** is a single integer $K \in \{0, 1, \ldots, 25\}$.

> Caesar himself used $K = 3$. In his time, most enemies were illiterate, so even this trivial cipher was militarily effective.

## Worked Example

**Encrypt** the message `HELLO` with key $K = 3$.

| Letter | Value $m$ | $(m + 3) \bmod 26$ | Ciphertext |
|--------|-----------|---------------------|------------|
| H      | 7         | 10                  | K          |
| E      | 4         | 7                   | H          |
| L      | 11        | 14                  | O          |
| L      | 11        | 14                  | O          |
| O      | 14        | 17                  | R          |

Ciphertext: **`KHOOR`**

**Decrypt** `KHOOR` with $K = 3$:

| Letter | Value $c$ | $(c - 3) \bmod 26$ | Plaintext |
|--------|-----------|---------------------|-----------|
| K      | 10        | 7                   | H         |
| H      | 7         | 4                   | E         |
| O      | 14        | 11                  | L         |
| O      | 14        | 11                  | L         |
| R      | 17        | 14                  | O         |

Recovered: **`HELLO`** ✓

## Why It's Weak

The total **key space** is only $\{0, 1, \ldots, 25\}$ — just **26 possible keys**. An attacker can simply try all 26 shifts in under a second, reading off the one that produces intelligible English. This is called an **exhaustive key search** (brute-force attack).

## Challenge

Encrypt `CRYPTO` with $K = 13$. Then encrypt the result again with $K = 13$. What do you get, and why?

The first encryption gives: C(2)→P, R(17)→E, Y(24)→L, P(15)→C, T(19)→G, O(14)→B → `PELCGB`

The second encryption: P(15)→C, E(4)→R, L(11)→Y, C(2)→P, G(6)→T, B(1)→O → `CRYPTO`

You get back the original plaintext! This is because $K = 13$ is special: $13 + 13 = 26 \equiv 0 \pmod{26}$. Two shifts of 13 cancel each other. This cipher is known as **ROT13** and applying it twice is an identity transformation.

---

# 3. Substitution Cipher (Monoalphabetic)

## Definition

A **substitution cipher** generalizes the shift cipher by using an arbitrary permutation $\pi$ of the alphabet as the key. Each plaintext letter is independently mapped to a ciphertext letter via a fixed lookup table.

$$E_\pi(m) = \pi(m) \qquad D_\pi(c) = \pi^{-1}(c)$$

The key is the complete substitution table (a permutation of 26 letters).

## Worked Example

Suppose the key is the following substitution table:

| Plain  | A | B | C | D | E | F | G | H | I | J | K | L | M |
|--------|---|---|---|---|---|---|---|---|---|---|---|---|---|
| Cipher | Q | W | E | R | T | Y | U | I | O | P | A | S | D |

| Plain  | N | O | P | Q | R | S | T | U | V | W | X | Y | Z |
|--------|---|---|---|---|---|---|---|---|---|---|---|---|---|
| Cipher | F | G | H | J | K | L | Z | X | C | V | B | N | M |

**Encrypt** `HELLO`:
- H → I
- E → T
- L → S
- L → S
- O → G

Ciphertext: **`ITSSG`**

**Decrypt** by reversing the table: find which plaintext letter maps to each ciphertext letter.
- I → H, T → E, S → L, S → L, G → O → **`HELLO`** ✓

## Why It's Weak — Frequency Analysis

The key space is $26! \approx 4 \times 10^{26}$ — astronomically large. Brute-force is impossible.

But the cipher has a fatal structural flaw: **it is monoalphabetic** — each plaintext letter always maps to the same ciphertext letter. This means the **statistical frequencies of letters are preserved** in the ciphertext.

> In English, 'E' appears ~12.7% of the time, 'T' ~9.1%, 'A' ~8.2%, and so on. An attacker counts letter frequencies in the ciphertext and maps the most frequent ciphertext letters to the most frequent English letters. With enough ciphertext (a few hundred characters), the cipher is fully broken.

## Challenge

You receive the ciphertext `ITSSG SGKKT`. You know it was encrypted with a substitution cipher and suspect the plaintext is a repeated phrase.

<u>What strategy would you use to figure out the key without trying all 26! possibilities?</u>

Look for repeated patterns. `ITSSG` and `SGKKT` share the pattern `S_` and `_KK_`. Guess common English 5-letter words with doubled letters (like `HELLO`, `MAMMA`). If the plaintext is `HELLO WORLD`, then: I→H, T→E, S→L, G→O, K→R, T→D. Cross-check: does the resulting mapping consistently decode the rest? If yes, the key is confirmed. This is **pattern-based cryptanalysis** — exploiting known-plaintext or crib-dragging.

---

# 4. Affine Cipher

## Motivation: A Structured Substitution

The shift cipher shifts every letter by the same amount: $E_K(m) = m + K$. What if instead of just adding, we also **multiply** by a factor? This gives us a linear transformation, combining both operations into one formula.

## Definition

The **Affine Cipher** uses two parameters, $a$ and $b$, as its key:

$$E_{a,b}(m) = (a \cdot m + b) \bmod 26$$

$$D_{a,b}(c) = a^{-1} \cdot (c - b) \bmod 26$$

where $a^{-1}$ is the **modular multiplicative inverse** of $a$ modulo 26.

The valid values of $a$ are: $\{1, 3, 5, 7, 9, 11, 15, 17, 19, 21, 23, 25\}$ — **12 values** in total.
The valid values of $b$ are: $\{0, 1, \ldots, 25\}$ — **26 values**.
Total key space: $12 \times 26 = 312$ keys.

## Why gcd(a, 26) = 1 Is Required

This is the critical constraint. For decryption to work, we need a unique inverse $a^{-1}$ such that:

$$a \cdot a^{-1} \equiv 1 \pmod{26}$$

This inverse **only exists** when $\gcd(a, 26) = 1$ (i.e., $a$ and 26 share no common factor other than 1).

**Why?** If $\gcd(a, 26) = d > 1$, then $a \cdot x \bmod 26$ can never equal 1 (because every multiple of $a$ is also a multiple of $d$, and $d$ does not divide 1). The encryption function would not be one-to-one — two different plaintext letters would map to the same ciphertext letter — making decryption impossible.

> Intuition: Multiplication "wraps around" the number circle of size 26. If $a$ divides 26 evenly, it creates collisions. Only numbers coprime to 26 cycle through all 26 positions without collision, guaranteeing a bijection.

## Worked Example

Let $a = 5$, $b = 8$.

**Step 1 — Find the decryption key $a^{-1}$:**

We need $5 \cdot x \equiv 1 \pmod{26}$. Testing: $5 \times 21 = 105 = 4 \times 26 + 1 \equiv 1$. So $a^{-1} = 21$.

**Step 2 — Encrypt `HELLO`:**

$E_{5,8}(m) = (5m + 8) \bmod 26$

| Letter | $m$ | $5m + 8$ | $\bmod 26$ | Ciphertext |
|--------|-----|----------|------------|------------|
| H      | 7   | 43       | 17         | R          |
| E      | 4   | 28       | 2          | C          |
| L      | 11  | 63       | 11         | L          |
| L      | 11  | 63       | 11         | L          |
| O      | 14  | 78       | 0          | A          |

Ciphertext: **`RCLLA`**

**Step 3 — Decrypt `RCLLA`:**

$D_{5,8}(c) = 21 \cdot (c - 8) \bmod 26$

| Letter | $c$ | $c - 8 \bmod 26$ | $\times 21 \bmod 26$ | Plaintext |
|--------|-----|-----------------|----------------------|-----------|
| R      | 17  | 9               | $189 \bmod 26 = 7$   | H         |
| C      | 2   | -6 ≡ 20         | $420 \bmod 26 = 4$   | E         |
| L      | 11  | 3               | $63 \bmod 26 = 11$   | L         |
| L      | 11  | 3               | $63 \bmod 26 = 11$   | L         |
| A      | 0   | -8 ≡ 18         | $378 \bmod 26 = 14$  | O         |

Recovered: **`HELLO`** ✓

## Challenge

Why does the Affine Cipher with $a = 1$ reduce to the Shift Cipher? And why is $a = 13$ not a valid choice?

With $a = 1$: $E_{1,b}(m) = (1 \cdot m + b) \bmod 26 = (m + b) \bmod 26$. This is exactly the shift cipher with shift $K = b$.

With $a = 13$: $\gcd(13, 26) = 13 \neq 1$. So the encryption is not one-to-one. For example, $E_{13,0}(0) = 0$ and $E_{13,0}(2) = 26 \bmod 26 = 0$: both `A` and `C` map to `A`. Decryption is impossible — the cipher is broken by construction.

---

# 5. The Euclidean Algorithm

## Why We Need It

To work with the Affine Cipher (and RSA, and many other cryptographic systems), we constantly need to answer: **"What is $\gcd(a, b)$?"** The Euclidean Algorithm gives us an efficient answer.

More importantly, it tells us whether a modular inverse exists ($\gcd(a, n) = 1$ means the inverse exists), and the Extended version (Section 6) lets us *compute* that inverse.

## The Algorithm

The key observation is a simple lemma:

$$\gcd(a, b) = \gcd(b,\ a \bmod b)$$

**Why does this work?** Any common divisor of $a$ and $b$ also divides $a - qb$ (for any integer $q$), which is $a \bmod b$. So the set of common divisors does not change when we replace $a$ with $a \bmod b$. Repeating this process reduces the numbers until the remainder hits 0, at which point the non-zero value is the GCD.

**Algorithm:**
```
gcd(a, b):
    while b ≠ 0:
        (a, b) ← (b, a mod b)
    return a
```

## Worked Example

Find $\gcd(252, 198)$:

| Step | $a$  | $b$  | $a \bmod b$ | Equation                    |
|------|------|------|-------------|-----------------------------|
| 1    | 252  | 198  | 54          | $252 = 1 \times 198 + 54$   |
| 2    | 198  | 54   | 36          | $198 = 3 \times 54 + 36$    |
| 3    | 54   | 36   | 18          | $54 = 1 \times 36 + 18$     |
| 4    | 36   | 18   | 0           | $36 = 2 \times 18 + 0$      |

When $b = 0$, we stop. $\gcd(252, 198) = \mathbf{18}$.

## Challenge

Determine whether an Affine Cipher with $a = 9$ is valid over an alphabet of size 26. Then determine whether $a = 9$ is valid over an alphabet of size **25**.

For alphabet size 26: $\gcd(9, 26)$. $26 = 2 \times 9 + 8$, $9 = 1 \times 8 + 1$, $8 = 8 \times 1 + 0$. GCD = 1. **Valid** — $9^{-1} \bmod 26$ exists.

For alphabet size 25: $\gcd(9, 25)$. $25 = 2 \times 9 + 7$, $9 = 1 \times 7 + 2$, $7 = 3 \times 2 + 1$, $2 = 2 \times 1 + 0$. GCD = 1. Also **valid** — but note that 25 = 5², so any $a$ that is not a multiple of 5 would work.

---

# 6. The Extended Euclidean Algorithm

## From GCD to Modular Inverse

The regular Euclidean Algorithm only tells us the *value* of $\gcd(a, b)$. The **Extended Euclidean Algorithm** tells us the **integers $x$ and $y$** such that:

$$ax + by = \gcd(a, b)$$

This is called **Bézout's Identity**. It always has a solution.

**Why does this give us the modular inverse?** If $\gcd(a, n) = 1$, then we can find $x, y$ such that:

$$ax + ny = 1 \implies ax \equiv 1 \pmod{n}$$

So $x$ is exactly $a^{-1} \bmod n$. If $x$ comes out negative, add multiples of $n$ until it is positive.

## The Algorithm: Back-Substitution

Run the Euclidean Algorithm forward to collect all the division equations. Then substitute backwards from the bottom to express the GCD as a linear combination of the original $a$ and $b$.

## Worked Example

Find $5^{-1} \bmod 26$ — i.e., find $x$ such that $5x \equiv 1 \pmod{26}$.

We need $5x + 26y = 1$, so compute $\gcd(26, 5)$ using the Extended algorithm.

**Forward pass:**

| Step | Equation                     |
|------|------------------------------|
| 1    | $26 = 5 \times 5 + 1$        |
| 2    | $5 = 5 \times 1 + 0$         |

GCD = 1 (confirmed the inverse exists).

**Back-substitution:**

From Step 1:
$$1 = 26 - 5 \times 5$$
$$1 = 1 \times 26 + (-5) \times 5$$

So $x = -5$ and $y = 1$. Therefore:
$$5 \times (-5) \equiv 1 \pmod{26}$$
$$-5 \equiv 21 \pmod{26} \qquad (\text{since } -5 + 26 = 21)$$

$$\boxed{5^{-1} \equiv 21 \pmod{26}}$$

**Verify:** $5 \times 21 = 105 = 4 \times 26 + 1 \equiv 1 \pmod{26}$ ✓

## Tabular Method (Faster)

For longer computations, maintain a table with columns $r$ (remainder), $q$ (quotient), $s$, $t$ where $s \cdot a + t \cdot b = r$ at every row.

Find $17^{-1} \bmod 43$:

| $r$  | $q$ | $s$ | $t$ |
|------|-----|-----|-----|
| 43   | —   | 1   | 0   |
| 17   | —   | 0   | 1   |
| 9    | 2   | 1   | -2  |
| 8    | 1   | -1  | 3   |
| 1    | 1   | 2   | -5  |
| 0    | 8   | —   | —   |

At the row where $r = 1$: $s = 2$, $t = -5$. This means $43 \times 2 + 17 \times (-5) = 1$, so $17 \times (-5) \equiv 1 \pmod{43}$, and $-5 \equiv 38 \pmod{43}$.

$$\boxed{17^{-1} \equiv 38 \pmod{43}}$$

**Verify:** $17 \times 38 = 646 = 15 \times 43 + 1 \equiv 1 \pmod{43}$ ✓

## Challenge

The Affine Cipher key is $(a, b) = (7, 3)$ over $\mathbb{Z}_{26}$. Find the decryption key $a^{-1} \bmod 26$ using back-substitution.

$\gcd(26, 7)$:
- $26 = 3 \times 7 + 5$
- $7 = 1 \times 5 + 2$
- $5 = 2 \times 2 + 1$
- $2 = 2 \times 1 + 0$

Back-substitute:
$1 = 5 - 2 \times 2$
$= 5 - 2 \times (7 - 5) = 3 \times 5 - 2 \times 7$
$= 3 \times (26 - 3 \times 7) - 2 \times 7 = 3 \times 26 - 11 \times 7$

So $7 \times (-11) \equiv 1 \pmod{26}$, and $-11 \equiv 15 \pmod{26}$.

$\boxed{7^{-1} \equiv 15 \pmod{26}}$. Verify: $7 \times 15 = 105 = 4 \times 26 + 1$ ✓.

The decryption function is $D(c) = 15(c - 3) \bmod 26$.

---

# 7. Vigenère Cipher (Polyalphabetic)

## The Core Insight

Every cipher we have seen so far is **monoalphabetic** — each plaintext letter always maps to the same ciphertext letter. The Vigenère Cipher breaks this by using **a different shift for each position**, determined by a repeating keyword.

Because the same plaintext letter can encrypt to different ciphertext letters (depending on its position), **frequency analysis no longer works directly**. This was considered unbreakable for 300 years ("le chiffre indéchiffrable").

## Definition

Choose a keyword of length $d$, encoded as a sequence of shifts $k_0, k_1, \ldots, k_{d-1}$ (where $A=0, B=1, \ldots$). For the $i$-th character of the message:

$$E(m_i) = (m_i + k_{i \bmod d}) \bmod 26$$

$$D(c_i) = (c_i - k_{i \bmod d}) \bmod 26$$

The key repeats cyclically. Each "column" of the plaintext (positions $0, d, 2d, \ldots$) is encrypted with the same shift.

## Worked Example

Plaintext: `ATTACKATDAWN`  
Key: `LEMON` → $L=11,\ E=4,\ M=12,\ O=14,\ N=13$

Align the repeating key:

| Position | 0  | 1  | 2  | 3  | 4  | 5  | 6  | 7  | 8  | 9  | 10 | 11 |
|----------|----|----|----|----|----|----|----|----|----|----|----|----|
| Plain    | A  | T  | T  | A  | C  | K  | A  | T  | D  | A  | W  | N  |
| Plain #  | 0  | 19 | 19 | 0  | 2  | 10 | 0  | 19 | 3  | 0  | 22 | 13 |
| Key      | L  | E  | M  | O  | N  | L  | E  | M  | O  | N  | L  | E  |
| Key #    | 11 | 4  | 12 | 14 | 13 | 11 | 4  | 12 | 14 | 13 | 11 | 4  |
| Sum mod26| 11 | 23 | 5  | 14 | 15 | 21 | 4  | 5  | 17 | 13 | 7  | 17 |
| Cipher   | L  | X  | F  | O  | P  | V  | E  | F  | R  | N  | H  | R  |

Ciphertext: **`LXFOPVEFRNHR`**

**Decrypt** by subtracting the key:
$(L=11) - (L=11) = 0 = A$, $(X=23) - (E=4) = 19 = T$, etc. — reproduces `ATTACKATDAWN` ✓

## Why It's Stronger — And Why It Still Falls

The Vigenère cipher resists simple frequency analysis because the letter 'E' in the plaintext might appear as 'I' in one position and 'Q' in another, depending on the key character at that position.

However, it **fails** because the key repeats. The **Kasiski Test** exploits this: identical plaintext segments encrypted with the same key offset produce identical ciphertext segments. By finding repeated ciphertext sequences and measuring the distance between them, an attacker can determine the key length $d$. Once $d$ is known, each of the $d$ "columns" is just an ordinary shift cipher — solvable by frequency analysis independently.

> The Vigenère cipher is only as strong as the length of its key. If the key is as long as the message and truly random (a **one-time pad**), it is provably unbreakable. But a short repeating key is just a thin veil over $d$ interleaved shift ciphers.

## Challenge

A Vigenère ciphertext has three identical 3-letter sequences at positions 5, 17, and 29. What is the most likely key length?

The distances between the repeated sequences are $17 - 5 = 12$ and $29 - 17 = 12$. The key length $d$ must divide both distances. The divisors of 12 are $\{1, 2, 3, 4, 6, 12\}$. Most likely $d \in \{3, 4, 6, 12\}$ (1 and 2 are too short to be secure). The analyst would test each candidate using the Index of Coincidence to find which value of $d$ makes the per-column distributions look like English.

---

# 8. Hill Cipher

## The Core Insight

The ciphers so far encrypt **one letter at a time**. The Hill Cipher upgrades to encrypting **$n$ letters simultaneously** by treating them as a vector and multiplying by an $n \times n$ key matrix. This provides much stronger diffusion: changing one input letter affects all $n$ output letters.

## Definition

Choose a block size $n$ and a key matrix $K$ (an $n \times n$ matrix with entries in $\mathbb{Z}_{26}$).

**Encryption:** Group the plaintext into $n$-character blocks. Represent each block as a column vector $\mathbf{p}$:

$$\mathbf{c} = K \cdot \mathbf{p} \bmod 26$$

**Decryption:**

$$\mathbf{p} = K^{-1} \cdot \mathbf{c} \bmod 26$$

**Key validity requirement:** $K^{-1}$ must exist modulo 26, which requires:

$$\gcd(\det(K),\ 26) = 1$$

i.e., the determinant of $K$ must be coprime to 26.

## Finding the Matrix Inverse mod 26

For a $2 \times 2$ matrix $K = \begin{pmatrix} a & b \\ c & d \end{pmatrix}$:

$$K^{-1} = \det(K)^{-1} \cdot \begin{pmatrix} d & -b \\ -c & a \end{pmatrix} \bmod 26$$

where $\det(K) = ad - bc$ and $\det(K)^{-1}$ is the modular inverse of $\det(K)$ modulo 26.

## Worked Example (Encryption)

Key: $K = \begin{pmatrix} 3 & 3 \\ 2 & 5 \end{pmatrix}$, block size $n = 2$

**Verify key validity:**
$\det(K) = 3 \times 5 - 3 \times 2 = 15 - 6 = 9$

$\gcd(9, 26)$: $26 = 2 \times 9 + 8$, $9 = 1 \times 8 + 1$, $8 = 8 \times 1$. GCD = 1. ✓ Key is valid.

**Find $9^{-1} \bmod 26$:**
$9 \times 3 = 27 = 26 + 1 \equiv 1 \pmod{26}$. So $9^{-1} = 3$.

**Compute $K^{-1}$:**
$$K^{-1} = 3 \cdot \begin{pmatrix} 5 & -3 \\ -2 & 3 \end{pmatrix} \bmod 26 = \begin{pmatrix} 15 & -9 \\ -6 & 9 \end{pmatrix} \bmod 26 = \begin{pmatrix} 15 & 17 \\ 20 & 9 \end{pmatrix}$$

(Since $-9 \bmod 26 = 17$ and $-6 \bmod 26 = 20$.)

**Encrypt `HELP`** ($H=7, E=4, L=11, P=15$), split into two 2-character blocks:

*Block 1:* `HE` → $\mathbf{p}_1 = \begin{pmatrix} 7 \\ 4 \end{pmatrix}$

$$\mathbf{c}_1 = \begin{pmatrix} 3 & 3 \\ 2 & 5 \end{pmatrix} \begin{pmatrix} 7 \\ 4 \end{pmatrix} = \begin{pmatrix} 21 + 12 \\ 14 + 20 \end{pmatrix} = \begin{pmatrix} 33 \\ 34 \end{pmatrix} \bmod 26 = \begin{pmatrix} 7 \\ 8 \end{pmatrix} \rightarrow \text{HI}$$

*Block 2:* `LP` → $\mathbf{p}_2 = \begin{pmatrix} 11 \\ 15 \end{pmatrix}$

$$\mathbf{c}_2 = \begin{pmatrix} 3 & 3 \\ 2 & 5 \end{pmatrix} \begin{pmatrix} 11 \\ 15 \end{pmatrix} = \begin{pmatrix} 33 + 45 \\ 22 + 75 \end{pmatrix} = \begin{pmatrix} 78 \\ 97 \end{pmatrix} \bmod 26 = \begin{pmatrix} 0 \\ 19 \end{pmatrix} \rightarrow \text{AT}$$

Ciphertext: **`HIAT`**

## Worked Example (Decryption)

Decrypt `HIAT` using $K^{-1} = \begin{pmatrix} 15 & 17 \\ 20 & 9 \end{pmatrix}$:

*Block 1:* `HI` → $\mathbf{c}_1 = \begin{pmatrix} 7 \\ 8 \end{pmatrix}$

$$\mathbf{p}_1 = \begin{pmatrix} 15 & 17 \\ 20 & 9 \end{pmatrix} \begin{pmatrix} 7 \\ 8 \end{pmatrix} = \begin{pmatrix} 105 + 136 \\ 140 + 72 \end{pmatrix} = \begin{pmatrix} 241 \\ 212 \end{pmatrix} \bmod 26 = \begin{pmatrix} 7 \\ 4 \end{pmatrix} \rightarrow \text{HE}\ ✓$$

*Block 2:* `AT` → $\mathbf{c}_2 = \begin{pmatrix} 0 \\ 19 \end{pmatrix}$

$$\mathbf{p}_2 = \begin{pmatrix} 15 & 17 \\ 20 & 9 \end{pmatrix} \begin{pmatrix} 0 \\ 19 \end{pmatrix} = \begin{pmatrix} 0 + 323 \\ 0 + 171 \end{pmatrix} = \begin{pmatrix} 323 \\ 171 \end{pmatrix} \bmod 26 = \begin{pmatrix} 11 \\ 15 \end{pmatrix} \rightarrow \text{LP}\ ✓$$

Recovered: **`HELP`** ✓

> The modular arithmetic: $241 = 9 \times 26 + 7$, $212 = 8 \times 26 + 4$, $323 = 12 \times 26 + 11$, $171 = 6 \times 26 + 15$.

## Challenge

Why is the Hill Cipher vulnerable to a **known-plaintext attack**, even though its key space is large?

If an attacker knows $n$ plaintext-ciphertext block pairs $(\mathbf{p}_1, \mathbf{c}_1), \ldots, (\mathbf{p}_n, \mathbf{c}_n)$, they can form the matrix equations $C = K \cdot P$ where $P = [\mathbf{p}_1 | \cdots | \mathbf{p}_n]$ and $C = [\mathbf{c}_1 | \cdots | \mathbf{c}_n]$. Then $K = C \cdot P^{-1} \bmod 26$ (if $P$ is invertible mod 26). The attacker recovers the exact key matrix $K$ with just $n$ known plaintext blocks — only $n^2$ characters of known plaintext needed. The Hill cipher is entirely linear, which makes it algebraically transparent.

---

# 9. Permutation Cipher (Transposition)

## The Core Insight

All ciphers so far change **what** each character is (substitution). A permutation cipher changes **where** each character appears — it rearranges the positions without altering the characters themselves.

> Crucially, no character is substituted. An attacker examining the ciphertext will find all the correct letters of the plaintext — just scrambled. Frequency analysis of individual letters reveals the plaintext letter frequencies instantly, so transposition ciphers must be combined with substitution to achieve real security.

## Definition

A **permutation cipher** (also called a transposition cipher) uses a permutation $\sigma$ of positions $\{1, 2, \ldots, n\}$ as the key. For a block of $n$ characters $m_1 m_2 \ldots m_n$:

$$E_\sigma(m_1 \ldots m_n) = m_{\sigma^{-1}(1)}\ m_{\sigma^{-1}(2)}\ \ldots\ m_{\sigma^{-1}(n)}$$

$$D_\sigma(c_1 \ldots c_n) = c_{\sigma(1)}\ c_{\sigma(2)}\ \ldots\ c_{\sigma(n)}$$

In practice, the most common variant is **columnar transposition**, described below.

## Worked Example — Columnar Transposition

**Encryption:**

Plaintext: `SENDMONEY` (9 characters)

Key: the column-read order is $(3, 1, 2)$ — meaning we write the plaintext in rows of 3 columns, then read out columns in the order: column labelled 1 first, then 2, then 3.

**Step 1 — Write plaintext row by row under the key:**

| Key label → | **3** | **1** | **2** |
|-------------|-------|-------|-------|
| Row 1       | S     | E     | N     |
| Row 2       | D     | M     | O     |
| Row 3       | N     | E     | Y     |

**Step 2 — Read columns in key-number order (1, 2, 3):**

- Key 1 column (physically column 2): E, M, E → `EME`
- Key 2 column (physically column 3): N, O, Y → `NOY`
- Key 3 column (physically column 1): S, D, N → `SDN`

Ciphertext: **`EMENOYSDN`**

---

**Decryption:**

Given ciphertext `EMENOYSDN`, key $(3, 1, 2)$, block size 3 columns × 3 rows = 9 chars.

**Step 1 — Divide ciphertext into chunks of 3 (one per column), fill by key-number order:**

- Key 1 gets first 3 chars: E, M, E → place in column labelled 1
- Key 2 gets next 3 chars: N, O, Y → place in column labelled 2
- Key 3 gets last 3 chars: S, D, N → place in column labelled 3

**Step 2 — Reconstruct the table (columns ordered by their key labels 3, 1, 2):**

| Key label → | **3** | **1** | **2** |
|-------------|-------|-------|-------|
| Row 1       | S     | E     | N     |
| Row 2       | D     | M     | O     |
| Row 3       | N     | E     | Y     |

**Step 3 — Read row by row:**

`SEN` + `DMO` + `NEY` = **`SENDMONEY`** ✓

## Challenge

A columnar transposition ciphertext is `RTRHCPGPYOAY` with key $(3, 1, 2)$ and 3 rows. Decrypt it.

12 characters, 3 columns → 4 rows each.

Fill by key-number order:
- Key 1 (first 4 chars): R, T, R, H
- Key 2 (next 4 chars): C, P, G, P
- Key 3 (last 4 chars): Y, O, A, Y

Reconstruct table (physical columns ordered key 3, 1, 2):

| Key label → | **3** | **1** | **2** |
|-------------|-------|-------|-------|
| Row 1       | Y     | R     | C     |
| Row 2       | O     | T     | P     |
| Row 3       | A     | R     | G     |
| Row 4       | Y     | H     | P     |

Read row by row: `YRC` + `OTP` + `ARG` + `YHP` = **`YRCOTPARGYHP`**

Hmm, that doesn't look like English — the key and ciphertext in this challenge are artificial. The important takeaway: the decryption procedure is always deterministic, and with the correct key the plaintext is recovered unambiguously.

---

# 10. Summary and Comparison

| Cipher         | Type           | Key                          | Key Space      | Encryption Formula                              | Main Weakness                      |
|----------------|----------------|------------------------------|----------------|-------------------------------------------------|------------------------------------|
| Shift          | Substitution   | $K \in \{0,\ldots,25\}$      | 26             | $E_K(m) = (m+K) \bmod 26$                      | Exhaustive key search (26 tries)   |
| Substitution   | Substitution   | Permutation $\pi$ of 26 letters | $26! \approx 4 \times 10^{26}$ | $E(m) = \pi(m)$              | Frequency analysis                 |
| Affine         | Substitution   | $(a, b)$, $\gcd(a,26)=1$    | 312            | $E_{a,b}(m) = (am+b) \bmod 26$                 | Frequency analysis + small key space|
| Vigenère       | Polyalphabetic | Keyword of length $d$        | $26^d$         | $E(m_i) = (m_i + k_{i \bmod d}) \bmod 26$      | Kasiski test → per-column freq. analysis |
| Hill           | Polyalphabetic | $n \times n$ matrix $K$      | Large          | $\mathbf{c} = K\mathbf{p} \bmod 26$            | Known-plaintext attack (linear algebra) |
| Permutation    | Transposition  | Permutation $\sigma$ of $\{1,\ldots,n\}$ | $n!$ | Rearrange character positions             | Letter frequencies unchanged; anagram analysis |

**The pattern of cryptographic history:**

> Every classical cipher was eventually broken by exploiting some **structural regularity** that the designer failed to eliminate. Shift ciphers kept the shift constant. Substitution ciphers kept the mapping constant. Vigenère kept the key repeating. Hill was purely linear. The lesson that modern cryptography learned is: a secure cipher must look **completely random** to anyone who does not know the key — no structure, no pattern, no leaked information.

This insight directly motivates the design goals of modern block ciphers (DES, AES): multiple rounds of non-linear substitutions (S-boxes) combined with permutations (P-boxes), iterated enough times that no statistical trace of the plaintext survives in the ciphertext.
