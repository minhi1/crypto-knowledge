# 1. The Mathematical Foundation

RSA is built on top of several deep ideas from number theory. Before touching the cipher itself, you need to own these concepts — the cipher is just an application of the math.

## Modular Arithmetic

You already use modular arithmetic every day. A clock only has 12 positions. After 12, it wraps back to 1. The "hours" world is arithmetic modulo 12.

Formally: we say $a \equiv b \pmod{n}$ if $a$ and $b$ leave the same remainder when divided by $n$. Equivalently, $n$ divides $(a - b)$.

$$17 \equiv 5 \pmod{12} \quad \text{because } 17 - 5 = 12, \text{ divisible by } 12\\
38 \equiv 2 \pmod{9} \quad \text{because } 38 = 4 \times 9 + 2$$

The key operations all work modulo $n$:
- **Addition**: $(a + b) \bmod n$
- **Multiplication**: $(a \times b) \bmod n$
- **Exponentiation**: $a^k \bmod n$ — computed efficiently using *fast exponentiation* (repeated squaring)

The set $\{0, 1, 2, \ldots, n-1\}$ with these operations forms a mathematical structure called $\mathbb{Z}_n$ (integers mod $n$).

## Greatest Common Divisor and Coprimality

$\gcd(a, b)$ is the largest integer that divides both $a$ and $b$.

Two numbers are **coprime** (relatively prime) if $\gcd(a, b) = 1$ — they share no common factor other than 1.

$$\gcd(12, 35) = 1 \quad \text{(coprime — 12 = 4×3, 35 = 5×7, no shared primes)}$$
$$\gcd(12, 18) = 6 \quad \text{(not coprime)}$$

Coprimality matters for RSA because modular inverses only exist when numbers are coprime.

## Modular Inverse

The modular inverse of $a$ modulo $n$ is a number $a^{-1}$ such that:

$$a \cdot a^{-1} \equiv 1 \pmod{n}$$

This is the modular equivalent of division. Crucially, **$a^{-1}$ exists if and only if $\gcd(a, n) = 1$**. If $a$ and $n$ share a factor, there is no inverse.

**Example:** Find $3^{-1} \pmod{7}$.
Try: $3 \times 1 = 3$, $3 \times 2 = 6$, $3 \times 3 = 9 \equiv 2$, $3 \times 4 = 12 \equiv 5$, $3 \times 5 = 15 \equiv 1$. So $3^{-1} \equiv 5 \pmod{7}$.

For large numbers, the **Extended Euclidean Algorithm** computes this efficiently (covered in Section 4).

## Euler's Phi Function (Totient)

The **Euler phi function** (also called the totient function) $\phi(n)$ counts how many integers in $\{1, 2, \ldots, n\}$ are coprime to $n$.

$$\phi(n) = |\{k : 1 \leq k \leq n,\; \gcd(k, n) = 1\}|$$

Two critical formulas:

**For a prime $p$**: every integer from 1 to $p-1$ is coprime to $p$ (since $p$ has no divisors other than 1 and itself).
$$\phi(p) = p - 1$$

**For a product of two distinct primes $p$ and $q$**: the integers from 1 to $pq$ that are *not* coprime to $pq$ are exactly the multiples of $p$ (there are $q$ of them) and multiples of $q$ (there are $p$ of them), minus their overlap 0.

$$\phi(pq) = pq - q - p + 1 = (p-1)(q-1)$$

This formula is the engine of RSA. When $n = pq$, computing $\phi(n)$ is easy if you know $p$ and $q$, and hard if you don't.

## Euler's Theorem and Fermat's Little Theorem

**Fermat's Little Theorem**: If $p$ is prime and $\gcd(a, p) = 1$:
$$a^{p-1} \equiv 1 \pmod{p}$$

**Euler's Theorem** (the generalization): If $\gcd(a, n) = 1$:
$$a^{\phi(n)} \equiv 1 \pmod{n}$$

This is the theorem that makes RSA decryption work. If we raise $a$ to any multiple of $\phi(n)$ and add 1 back, we get $a$ again modulo $n$:

$$a^{k \cdot \phi(n) + 1} \equiv a \pmod{n}$$

---

# 2. RSA Key Generation

Now we can assemble the cipher. RSA key generation has five steps.

**Step 1: Choose two large distinct primes $p$ and $q$.**

In practice, each is 1024 bits long (giving a 2048-bit modulus). How to find them is covered in Section 7. Keep $p$ and $q$ completely secret — they are destroyed after key generation in many implementations.

**Step 2: Compute the modulus $n = p \times q$.**

$n$ is made public. The security of RSA rests on the fact that factoring $n$ back into $p$ and $q$ is computationally infeasible for large $n$.

**Step 3: Compute $\phi(n) = (p-1)(q-1)$.**

Keep $\phi(n)$ secret. Anyone who learns $\phi(n)$ can break the key.

**Step 4: Choose the public exponent $e$.**

Pick any $e$ satisfying:
- $1 < e < \phi(n)$
- $\gcd(e, \phi(n)) = 1$ (so the modular inverse of $e$ exists)

Common values: $e = 65537$ (= $2^{16} + 1$, chosen for fast exponentiation and well-studied security), or $e = 3$ (fast but risky — see Section 6).

**Step 5: Compute the private exponent $d$.**

Find $d$ such that:
$$e \cdot d \equiv 1 \pmod{\phi(n)}$$

$d$ is the modular inverse of $e$ modulo $\phi(n)$, computed via the Extended Euclidean Algorithm. Keep $d$ completely secret — this is the private key.

**Summary:**
- **Public key**: $(e,\; n)$
- **Private key**: $(d,\; n)$

Discard $p$, $q$, and $\phi(n)$ (or store them very carefully for Chinese Remainder Theorem acceleration).

---

# 3. RSA Encryption and Decryption

## Encryption

The sender looks up the recipient's public key $(e, n)$. The plaintext message $m$ must satisfy $0 \leq m < n$ (it is typically a padded representative of the actual message — see Section 6 on OAEP).

$$c = m^e \bmod n$$

## Decryption

The receiver applies their private key $(d, n)$:

$$m = c^d \bmod n$$

## Why Decryption Recovers the Plaintext

This is the elegant part. Decryption computes:
$$c^d = (m^e)^d = m^{ed} \pmod{n}$$

Since $e \cdot d \equiv 1 \pmod{\phi(n)}$, we know $ed = 1 + k \cdot \phi(n)$ for some integer $k$. Therefore:

$$m^{ed} = m^{1 + k \cdot \phi(n)} = m \cdot \left(m^{\phi(n)}\right)^k \equiv m \cdot 1^k = m \pmod{n}$$

The last step uses Euler's Theorem ($m^{\phi(n)} \equiv 1 \pmod{n}$ when $\gcd(m, n) = 1$). The chosen key pair $(e, d)$ effectively builds a trapdoor: exponentiation with $e$ scrambles the message, and only the matching $d$ (known only to the owner) can unscramble it.

## RSA as a Signature Scheme

RSA can also be run in reverse for **digital signatures**:
- The sender signs with their **private key**: $s = m^d \bmod n$
- Anyone verifies with the sender's **public key**: $m = s^e \bmod n$

Only the true holder of $d$ can produce a valid signature, but anyone with the public key can verify it.

---

# 4. The Extended Euclidean Algorithm

To compute $d = e^{-1} \bmod \phi(n)$, we use the Extended Euclidean Algorithm (EEA). The regular Euclidean Algorithm finds $\gcd(a, b)$ by repeated division. The *extended* version also finds integers $x$ and $y$ satisfying **Bézout's identity**:

$$ax + by = \gcd(a, b)$$

When $\gcd(e, \phi(n)) = 1$, this gives us:

$$e \cdot x + \phi(n) \cdot y = 1 \implies e \cdot x \equiv 1 \pmod{\phi(n)}$$

So $d = x \bmod \phi(n)$.

## Worked Example

Find $d = 3^{-1} \bmod 40$ (i.e., $e = 3$, $\phi(n) = 40$).

We run the Euclidean algorithm on $(40, 3)$ and back-substitute:

| Step | Equation | Quotient | Remainder |
|---|---|---|---|
| 1 | $40 = 13 \times 3 + 1$ | 13 | **1** |
| 2 | $3 = 3 \times 1 + 0$ | — | 0 → stop |

$\gcd(40, 3) = 1$ ✓. Now back-substitute to express 1 as a linear combination:

$$1 = 40 - 13 \times 3$$

So $x = -13$, giving $d = -13 \bmod 40 = 27$.

**Verification:** $3 \times 27 = 81 = 2 \times 40 + 1 \equiv 1 \pmod{40}$ ✓

---

# 5. Full Worked Example

Let's run RSA end-to-end with small numbers so every step is traceable.

**Key generation:**

Choose $p = 5$, $q = 11$.

$$n = 5 \times 11 = 55$$
$$\phi(n) = (5-1)(11-1) = 4 \times 10 = 40$$

Choose $e = 3$. Check: $\gcd(3, 40) = 1$ ✓

Find $d$: $3 \times d \equiv 1 \pmod{40}$. From the example above, $d = 27$.

- **Public key**: $(3,\; 55)$
- **Private key**: $(27,\; 55)$

**Encryption** of $m = 7$:

$$c = 7^3 \bmod 55 = 343 \bmod 55$$
$$343 = 6 \times 55 + 13 \implies c = 13$$

**Decryption** of $c = 13$:

$$m = 13^{27} \bmod 55$$

We use fast exponentiation (repeated squaring). Write $27 = 16 + 8 + 2 + 1$:

| Power | Computation | Value mod 55 |
|---|---|---|
| $13^1$ | $13$ | $13$ |
| $13^2$ | $13^2 = 169 = 3 \times 55 + 4$ | $4$ |
| $13^4$ | $4^2 = 16$ | $16$ |
| $13^8$ | $16^2 = 256 = 4 \times 55 + 36$ | $36$ |
| $13^{16}$ | $36^2 = 1296 = 23 \times 55 + 31$ | $31$ |

Now combine: $13^{27} = 13^{16} \times 13^8 \times 13^2 \times 13^1$

$$31 \times 36 = 1116 = 20 \times 55 + 16 \implies 16$$
$$16 \times 4 = 64 = 1 \times 55 + 9 \implies 9$$
$$9 \times 13 = 117 = 2 \times 55 + 7 \implies \mathbf{7}$$

$m = 7$ ✓ — the original plaintext is recovered.

---

# 6. Attacks on RSA

RSA's security depends critically on using it correctly. Many historical failures came not from breaking the math, but from misuse.

## The Core Hardness Assumption: Integer Factorization

The security of RSA reduces to the **integer factorization problem**: given $n = pq$, find $p$ and $q$.

If an attacker factors $n$, they immediately compute $\phi(n) = (p-1)(q-1)$, then run the EEA to recover $d$. The entire private key is exposed.

For a 2048-bit RSA modulus, the best known factoring algorithm would require more computation than all current computing power on Earth could perform in the age of the universe. This is why key length matters so much.

## Håstad's Broadcast Attack (Repeat Attack)

This is the canonical "repeat attack" — what happens when the same message is sent to multiple recipients.

Suppose a sender encrypts the same message $m$ using the same small public exponent $e = 3$ for three different recipients, each with their own modulus $n_1, n_2, n_3$:

$$c_1 = m^3 \bmod n_1$$
$$c_2 = m^3 \bmod n_2$$
$$c_3 = m^3 \bmod n_3$$

An eavesdropper collects all three ciphertexts and applies the **Chinese Remainder Theorem (CRT)**: since $n_1, n_2, n_3$ are pairwise coprime (they're products of different primes), there is a unique $M$ such that:

$$M \equiv c_1 \pmod{n_1}, \quad M \equiv c_2 \pmod{n_2}, \quad M \equiv c_3 \pmod{n_3}$$

and $M = m^3 \bmod (n_1 \cdot n_2 \cdot n_3)$.

Now if $m < \min(n_1, n_2, n_3)$ (which is true for any well-formed message), then $m^3 < n_1 \cdot n_2 \cdot n_3$, meaning the modular reduction never happened — $M = m^3$ exactly, as a plain integer. The attacker just takes the **integer cube root** of $M$ to recover $m$. No factoring required.

**The fix**: always use proper padding (OAEP) before encrypting. OAEP adds a fresh random value to every encryption, making the effective plaintext different for every recipient even if the original message $m$ is identical.

## Common Modulus Attack

If two users share the same modulus $n$ but have different exponents $(e_1, n)$ and $(e_2, n)$, and the same message $m$ is encrypted for both:

$$c_1 = m^{e_1} \bmod n, \quad c_2 = m^{e_2} \bmod n$$

If $\gcd(e_1, e_2) = 1$, the attacker uses the EEA to find $a, b$ such that $a \cdot e_1 + b \cdot e_2 = 1$, then computes:

$$c_1^a \cdot c_2^b \equiv m^{a \cdot e_1} \cdot m^{b \cdot e_2} = m^{a e_1 + b e_2} = m^1 = m \pmod{n}$$

The plaintext is recovered without factoring $n$ or knowing either private key.

**The fix**: never share a modulus between two different key pairs.

## Small Private Exponent Attack (Wiener's Attack)

Using a small $d$ (for fast decryption) is tempting but dangerous. **Wiener's theorem** (1990) proves that if $d < \frac{1}{3} n^{1/4}$, the private key $d$ can be efficiently recovered from the public key $(e, n)$ using **continued fractions**.

**The fix**: use $d$ of full size. The recommended mitigation is using CRT form for private key operations (which speeds up decryption by ~4× without reducing $d$).

## Small Public Exponent Attack

If $e$ is small (e.g., $e = 3$) and the plaintext $m$ is also small enough that $m^3 < n$, then $c = m^3$ with no modular reduction. The attacker just computes $\lfloor c^{1/3} \rfloor$.

This is a special case of the broadcast attack with only one recipient.

**The fix**: again, OAEP padding guarantees $m$ is randomized and fills the full modulus size, preventing this.

## Timing Attack

When a processor computes $m = c^d \bmod n$ using square-and-multiply, the time taken leaks information about $d$. Each bit of $d$ causes a slightly different number of modular multiplications, and by measuring decryption time across many ciphertexts, an attacker can recover $d$ bit by bit.

**The fix**: use blinding — before decryption, multiply $c$ by a random factor $r^e \bmod n$, decrypt, then divide out $r$. The timing no longer correlates to $d$.

---

# 7. The Prime Number Problem

Generating RSA keys requires two large primes $p$ and $q$. This raises a practical question: how do we find a 1024-bit prime?

## How Dense Are Primes?

The **Prime Number Theorem** tells us that the number of primes up to $N$ is approximately $N / \ln N$. The probability that a random number near $N$ is prime is approximately $1 / \ln N$.

For a 1024-bit number ($N \approx 2^{1024}$):
$$\ln(2^{1024}) = 1024 \ln 2 \approx 710$$

So roughly **1 in 710** random 1024-bit numbers is prime. That means if we randomly sample 1024-bit odd numbers and test each one, we'll find a prime after about 355 trials on average.

## The Strategy

1. Generate a random odd number $p$ of the required bit length.
2. Run a fast primality test.
3. If composite, try $p + 2$ (next odd number) or generate a fresh random number.
4. Repeat until a prime is found.

The only remaining question is: how do we efficiently test primality?

---

# 8. Primality Testing

## Naive Approach: Trial Division

Test whether $n$ is divisible by any integer from 2 up to $\sqrt{n}$.

- **Correct** — never gives a wrong answer
- **Hopelessly slow** for large numbers — $\sqrt{2^{1024}}$ is a $10^{154}$-digit number

Trial division is only used as a first filter (check small primes like 2, 3, 5, 7, 11, ..., up to a few thousand) to quickly eliminate obvious composites before the expensive tests.

## Fermat Primality Test

Fermat's Little Theorem guarantees that for a prime $p$:
$$a^{p-1} \equiv 1 \pmod{p} \quad \text{for any } a \text{ with } \gcd(a, p) = 1$$

**Test idea:** Pick a random $a$. Compute $a^{n-1} \bmod n$.
- If the result is $\neq 1$: $n$ is definitely **composite** — we found a *Fermat witness*.
- If the result is $= 1$: $n$ is **probably prime** — call $n$ a *Fermat pseudoprime* to base $a$.

Repeat with many random bases. If $n$ passes every test, it is probably prime.

**The fatal flaw: Carmichael numbers.** There exist composite numbers $n$ (called **Carmichael numbers**) that satisfy $a^{n-1} \equiv 1 \pmod{n}$ for *every* $a$ coprime to $n$. The smallest is $561 = 3 \times 11 \times 17$. The Fermat test will never detect these no matter how many bases you try.

## Miller-Rabin Primality Test

Miller-Rabin is a probabilistic test that is **immune to Carmichael numbers**. It is the standard algorithm used in every modern RSA implementation.

### Setup

Write $n - 1$ as $2^s \cdot d$ where $d$ is odd (factor out all powers of 2 from $n-1$).

**Example:** $n = 221$. $n - 1 = 220 = 4 \times 55 = 2^2 \times 55$. So $s = 2$, $d = 55$.

### One Round of the Test

Pick a random integer $a$ with $2 \leq a \leq n - 2$.

Compute $x = a^d \bmod n$.

**If** $x \equiv 1 \pmod{n}$ or $x \equiv -1 \pmod{n}$: this witness is inconclusive — $n$ *might* be prime. Move to the next witness.

**Otherwise**, repeatedly square $x$ up to $s - 1$ times:

For $i = 1, 2, \ldots, s-1$:
$$x \leftarrow x^2 \bmod n$$
- If $x \equiv -1 \pmod{n}$: inconclusive — move to the next witness.
- If $x \equiv 1 \pmod{n}$: **composite** — $n$ is definitely not prime.

If we complete all $s - 1$ squarings without seeing $-1$: **composite**.

If $n$ survives one round, repeat with a different random $a$.

### Why This Works

For a prime $p$, the sequence $a^d, a^{2d}, a^{4d}, \ldots, a^{2^s d} = a^{p-1}$ modulo $p$ must end in 1 (by Fermat). The only numbers whose square is 1 mod $p$ are $+1$ and $-1$ (since $p$ is prime, the equation $x^2 = 1$ has exactly 2 solutions). So the sequence must either:
- Start at 1, or
- Be $\ldots, -1, 1, 1, \ldots$ — i.e., hit $-1$ at some point before terminating at 1.

A composite $n$ can fool one random witness with probability **at most $1/4$**. This bound is tight but holds for all composites — unlike Fermat, which fails completely on Carmichael numbers.

### Error Probability

After $k$ independent rounds with $k$ random witnesses:

$$P(\text{n composite but passes all k rounds}) \leq \left(\frac{1}{4}\right)^k$$

| Rounds ($k$) | False-positive probability |
|---|---|
| 10 | $\approx 10^{-6}$ |
| 40 | $\approx 10^{-24}$ |
| 64 | $\approx 10^{-38}$ |

In practice, **40 rounds** is standard. The probability of error is so small it is dwarfed by the probability of random hardware failure.

### Full Worked Example

Test $n = 221$ for primality with witness $a = 174$.

**Step 0:** $n - 1 = 220 = 2^2 \times 55$. So $s = 2$, $d = 55$.

**Step 1:** Compute $x = 174^{55} \bmod 221$.

Using fast exponentiation: $174^{55} \bmod 221 = 47$.

$x = 47$. Is $47 \equiv 1$ or $47 \equiv 220$ (which is $-1 \bmod 221$)? Neither.

**Step 2:** $i = 1$: $x = 47^2 \bmod 221 = 2209 \bmod 221 = 2209 - 9 \times 221 = 2209 - 1989 = 220$.

$x = 220 \equiv -1 \pmod{221}$. This round is **inconclusive**.

Now try witness $a = 137$:

**Step 1:** $x = 137^{55} \bmod 221 = 188$.

$188 \neq 1$ and $188 \neq 220$.

**Step 2:** $i = 1$: $x = 188^2 \bmod 221 = 35344 \bmod 221 = 35344 - 159 \times 221 = 35344 - 35139 = 205$.

$205 \neq -1$ and $205 \neq 1$. We've exhausted all $s - 1 = 1$ squarings without finding $-1$.

**$n = 221$ is composite.** (Indeed, $221 = 13 \times 17$.)

### Deterministic Witnesses for Small $n$

For specific ranges of $n$, it is known which witnesses are *guaranteed* to give the correct answer — no randomness needed:

| Range | Witnesses that suffice |
|---|---|
| $n < 3{,}215{,}031{,}547$ | 2, 3, 5, 7 |
| $n < 3{,}317{,}044{,}064{,}679{,}887{,}385{,}961{,}981$ | 2, 3, 5, 7, 11, 13, 17, 19, 23, 29, 31, 37 |

For exam problems involving small numbers, you will almost always use just one or two witnesses.

---

# 9. Integer Factorization Algorithms

Security of RSA depends on factoring $n$ being hard. Understanding what "hard" means in practice requires knowing the actual algorithms.

## Trial Division

Test every prime up to $\sqrt{n}$. Works instantly for small numbers. For a 2048-bit RSA modulus, $\sqrt{n} \approx 2^{1024}$ — a number with over 300 digits. Completely infeasible.

## Pollard's Rho Algorithm

A probabilistic algorithm by John Pollard (1975). Uses Floyd's cycle-detection trick on a pseudo-random sequence modulo $n$.

**Expected running time:** $O(n^{1/4})$

For a 512-bit modulus (weak RSA from the 1990s), this is around $2^{128}$ — still impractical. But it's useful for finding small factors of large numbers. If $p$ is small (because someone chose a weak prime), Pollard's Rho finds it quickly. This is why RSA primes must both be large.

## General Number Field Sieve (GNFS)

The GNFS is the most powerful known algorithm for factoring large integers with no special structure. It operates in **sub-exponential** time:

$$O\!\left(\exp\!\left(\left(\frac{64}{9}\right)^{1/3} (\ln n)^{1/3} (\ln \ln n)^{2/3}\right)\right)$$

This grows much slower than exponential, but still faster than polynomial. For context:

| RSA Key Size | Estimated GNFS effort | Status |
|---|---|---|
| 512 bits | $\approx 2^{56}$ | Broken (1999, in 7 months) |
| 768 bits | $\approx 2^{76}$ | Broken (2009, ~2000 CPU-years) |
| 1024 bits | $\approx 2^{86}$ | Borderline — not recommended |
| 2048 bits | $\approx 2^{112}$ | Secure today |
| 4096 bits | $\approx 2^{140}$ | Long-term secure |

No algorithm better than GNFS is known for general RSA moduli. The gap between what attackers can do and what 2048-bit RSA requires is enormous.

---

# 10. Summary: RSA Security Requirements

For RSA to be secure in practice, all of the following must hold:

| Requirement | Why |
|---|---|
| $p$ and $q$ are both large (≥ 1024 bits each) | GNFS can factor if either prime is small |
| $p$ and $q$ are chosen independently at random | Common-factor attacks if primes share structure |
| $p \neq q$ and $p - q$ is large | Special factoring attacks when primes are close |
| $d$ is large ($d > n^{1/4}$) | Wiener's continued-fraction attack recovers small $d$ |
| Use OAEP padding | Prevents broadcast, small-exponent, and chosen-plaintext attacks |
| Never reuse $(e, n)$ across users | Common modulus attack |
| Never encrypt the same message to multiple recipients without padding | Håstad's broadcast attack |
| Blind private-key operations | Prevents timing side-channel attacks |
