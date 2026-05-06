# 1. What is a Hash Function?

A **cryptographic hash function** is a mathematical algorithm that takes an input of **arbitrary length** and produces a fixed-size output, called the **digest**, **hash value**, or simply **hash**.

$$H : \{0,1\}^* \;\longrightarrow\; \{0,1\}^n$$

The function maps any bit string — a short password, a 10 GB video, an entire hard drive — to exactly $n$ bits of output. The mapping is **deterministic**: the same input always produces the same output.

Think of it as a fingerprint machine. No matter how large the document, the machine always stamps a fixed-size fingerprint on it. Two different documents should (ideally) produce completely different fingerprints, and you should never be able to reconstruct the original document just from its fingerprint.

Hash functions appear everywhere in cryptography:
- **Data integrity**: verify a downloaded file wasn't corrupted or tampered with
- **Digital signatures**: sign $H(m)$ instead of $m$ directly (see DigitalSignature.md)
- **Password storage**: store $H(\text{password})$ so the server never holds the real password
- **Commitments**: commit to a value without revealing it
- **MACs**: build authenticated checksums with a secret key

---

# 2. Properties of Cryptographic Hash Functions

Not every hash function is cryptographically useful. A function like $H(m) = 0$ is fast and deterministic, but useless for security. Cryptographic hash functions must satisfy several precise security properties.

## 2.1 Pre-image Resistance (One-wayness)

> Given a hash output $h$, it is computationally infeasible to find **any** input $m$ such that $H(m) = h$.

In other words: the function is a one-way street. You can easily go from message to hash, but cannot go backwards. An attacker who intercepts a password hash $h = H(\text{password})$ cannot reverse-engineer the password.

**Attack cost**: brute-force requires trying all possible inputs — $O(2^n)$ attempts for an $n$-bit hash.

## 2.2 Second Pre-image Resistance (Weak Collision Resistance)

> Given a specific input $m_1$, it is computationally infeasible to find a **different** input $m_2 \neq m_1$ such that $H(m_1) = H(m_2)$.

This protects against an attacker who intercepts a signed document $m_1$ and tries to craft a different document $m_2$ with the same hash — so Alice's signature on $m_1$ also validates $m_2$.

**Attack cost**: brute-force requires $O(2^n)$ attempts.

## 2.3 Collision Resistance (Strong Collision Resistance)

> It is computationally infeasible to find **any two distinct inputs** $m_1 \neq m_2$ such that $H(m_1) = H(m_2)$.

This is stronger than second pre-image resistance because the attacker gets to choose **both** inputs freely, not just one. A pair $(m_1, m_2)$ with $H(m_1) = H(m_2)$ is called a **collision**.

**Attack cost**: because of the Birthday Paradox, collision search only requires $O(2^{n/2})$ attempts — much less than $O(2^n)$.

## 2.4 The Birthday Paradox: Why Collision Resistance is Harder

The Birthday Paradox states that in a group of just 23 people, there is a 50% chance two of them share a birthday. With only 70 people, the probability exceeds 99%. This seems counterintuitive because a year has 365 days.

The math: the probability of finding a collision among $k$ random samples from a space of $N$ values reaches 50% when $k \approx 1.177\sqrt{N}$.

Translated to hash functions with $n$-bit output ($N = 2^n$):

$$k \approx 1.177 \cdot 2^{n/2}$$

So for a 128-bit hash, finding a collision via random sampling takes about $2^{64}$ hashes — not $2^{128}$. This is why:
- **SHA-1** (160-bit output) has $2^{80}$ birthday-bound security, but was actually broken at $\approx 2^{63}$
- **SHA-256** (256-bit output) has $2^{128}$ birthday-bound collision security
- For a 128-bit hash to have 128-bit collision resistance, it would need 256-bit output

> **Rule of thumb**: an $n$-bit hash provides $n$ bits of security against pre-image attacks, but only $n/2$ bits of security against collision attacks. This is why modern standards use 256-bit or larger outputs.

## 2.5 Relationship Between the Three Properties

The properties form a hierarchy:

$$\text{Collision Resistance} \implies \text{Second Pre-image Resistance} \implies \text{Pre-image Resistance}$$

If a function is collision-resistant, it is also second-pre-image resistant (contrapositive: a second-pre-image attack is a special case of a collision). However, the reverse does not necessarily hold — a function could be second-pre-image resistant but still have collisions found via birthday attacks.

## 2.6 Avalanche Effect

A small change to the input — even flipping a single bit — should produce a completely different hash output. Ideally, each output bit changes with probability 50%.

**Example:** SHA-256 of "Hello" vs "hello" (one bit difference — capital H):
```
SHA-256("Hello") = 185f8db32921bd46d35cc848df34be...
SHA-256("hello") = 2cf24dba5fb0a30e26e83b2ac5b9e2...
```

The outputs share almost nothing in common, even though the inputs differ by a single bit. This is identical in spirit to the avalanche effect in block ciphers — and just as essential.

---

# 3. Classification: MDC and MAC

Cryptographic hash functions split into two families based on whether they use a **secret key**.

<div align="center">
<!-- ![Classification of hash functions](./images/hash-classification.png) -->
    <img src="./images/hash-classification.png">
</div>

## 3.1 MDC — Modification Detection Code (Unkeyed)

An MDC is a hash function with **no key**. Anyone can compute it. Its purpose is to detect accidental or malicious modification of data.

MDC is further divided:

- **One-Way Hash Function (OWHF)**: satisfies pre-image resistance and second pre-image resistance. Not necessarily collision-resistant.
- **Collision-Resistant Hash Function (CRHF)**: satisfies all three properties. This is what we mean by a "cryptographic hash function" in practice.

Examples of MDC: MD5, SHA-1, SHA-256, SHA-3.

**Key limitation of MDC**: because there is no secret key, an attacker who can intercept and replace both the message and its hash can substitute their own malicious message with a matching hash. MDC alone does not guarantee authenticity.

## 3.2 MAC — Message Authentication Code (Keyed)

A MAC is computed using both the message **and a shared secret key**. It provides:
- **Integrity**: detect any modification to the message
- **Authentication**: only someone who knows the key can produce a valid MAC

$$\text{MAC} = \text{Tag}(K,\; m)$$

Both sender and receiver share key $K$. The sender computes a tag and attaches it; the receiver independently computes the tag and compares.

**MAC vs Digital Signature:**
- MAC: symmetric — both parties need the key. Non-repudiation is impossible (the receiver could have forged the tag themselves).
- Digital Signature: asymmetric — only the signer has the private key. Non-repudiation is guaranteed.

### HMAC — Hash-Based MAC

HMAC wraps any MDC hash function to produce a keyed MAC. It was designed to be secure even if the underlying hash function has certain weaknesses.

$$\text{HMAC}(K,\; m) = H\!\bigl((K \oplus \text{opad}) \;\|\; H((K \oplus \text{ipad}) \;\|\; m)\bigr)$$

Where:
- $K$ is the key, zero-padded to the block size of the hash
- $\text{ipad}$ = `0x36` repeated to fill one block (inner padding)
- $\text{opad}$ = `0x5C` repeated to fill one block (outer padding)
- $\|$ denotes concatenation
- $H$ is the underlying hash function (e.g., SHA-256 → HMAC-SHA-256)

**Why two hashes?** The double application prevents **length-extension attacks**. If you only compute $H(K \| m)$, an attacker who knows $H(K \| m)$ can compute $H(K \| m \| m')$ for any suffix $m'$ without knowing $K$ — because Merkle-Damgård hash functions are vulnerable to this. The outer hash with `opad` closes this gap.

**Steps for HMAC-SHA-256:**
1. If $|K| >$ block size, set $K = H(K)$. Pad $K$ with zeros to block size.
2. Compute inner hash: $h_{\text{inner}} = H((K \oplus \text{ipad}) \| m)$
3. Compute outer hash: $\text{HMAC} = H((K \oplus \text{opad}) \| h_{\text{inner}})$

### CMAC — Cipher-Based MAC

CMAC (also known as OMAC) constructs a MAC from a block cipher rather than a hash function, using CBC mode:

1. Process the message block by block in CBC mode with a zero IV
2. Apply a special finalization step to the last block (using a derived subkey) to prevent extension attacks
3. The final CBC output is the authentication tag

CMAC-AES is a NIST standard (SP 800-38B) and is widely used in embedded systems and protocols like TLS.

### Comparison: HMAC vs CMAC vs a Raw Hash

| Property | Raw H(m) | HMAC | CMAC |
|---|---|---|---|
| Key required | No | Yes | Yes |
| Integrity | Yes | Yes | Yes |
| Authentication | No | Yes | Yes |
| Extension attack | Vulnerable | Immune | Immune |
| Underlying primitive | Hash | Hash | Block cipher |

---

# 4. MD5 — Message Digest 5

## Overview

MD5 was designed by Ron Rivest in 1991 as a successor to MD4. For over a decade it was the dominant hash function — used in TLS certificates, password storage, file integrity, and everywhere else.

**Parameters:**
- Output: 128 bits
- Internal state: 128 bits (4 × 32-bit words)
- Block size: 512 bits (processed as 16 × 32-bit little-endian words)
- Rounds: 4 rounds × 16 operations = **64 operations total**
- Word size: 32 bits, little-endian

**Current status: broken.** MD5 is completely compromised for security-sensitive uses. Collision attacks run in seconds on commodity hardware. Do not use MD5 for integrity or authentication.

## Step 1: Padding the Message

Before processing, the message is padded so its total length becomes congruent to 448 modulo 512 (in bits). The padding:

1. Append a single `1` bit to the end of the message
2. Append `0` bits until the length $\equiv 448 \pmod{512}$
3. Append the **original message length** (before padding) as a 64-bit **little-endian** integer

This guarantees the padded message is a multiple of 512 bits. The 64-bit length field is why MD5 (and SHA-1) can only process messages up to $2^{64} - 1$ bits.

**Note**: padding is always applied, even if the message is already 448 bits long — a full extra block is added.

## Step 2: Initialize the State Buffer

MD5 maintains four 32-bit state words, initialized to specific constants (little-endian byte order):

$$A = \texttt{0x67452301}$$
$$B = \texttt{0xEFCDAB89}$$
$$C = \texttt{0x98BADCFE}$$
$$D = \texttt{0x10325476}$$

These "magic numbers" are not arbitrary — written in little-endian byte order, they read as the byte sequences $01\,23\,45\,67$, $89\,AB\,CD\,EF$, $FE\,DC\,BA\,98$, $76\,54\,32\,10$.

## Step 3: The Four Auxiliary Functions

Each round of MD5 uses a different nonlinear function of three 32-bit inputs:

| Round | Function | Formula |
|---|---|---|
| 1 | $F(B, C, D)$ | $(B \wedge C) \vee (\neg B \wedge D)$ |
| 2 | $G(B, C, D)$ | $(B \wedge D) \vee (C \wedge \neg D)$ |
| 3 | $H(B, C, D)$ | $B \oplus C \oplus D$ |
| 4 | $I(B, C, D)$ | $C \oplus (B \vee \neg D)$ |

- $\wedge$ = AND, $\vee$ = OR, $\oplus$ = XOR, $\neg$ = NOT

$F$ is the **conditional select**: if $B$, take $C$; else take $D$ — identical to the AES S-box's underlying GF operation.

$H$ and the second use of $H$-like in round 4 are **parity-like XOR** mixers.

The different functions per round give each round different algebraic properties, making the cipher harder to analyze algebraically.

## Step 4: Processing Each 512-bit Block

For each 512-bit block of the padded message:

1. **Split the block** into 16 × 32-bit words: $M[0], M[1], \ldots, M[15]$ (little-endian)

2. **Initialize round variables**: $a = A,\; b = B,\; c = C,\; d = D$

3. **Run 64 operations** in 4 rounds of 16 each. Each operation $i$ (from 0 to 63) computes:

$$\text{temp} = a + \text{AuxFunc}(b, c, d) + M[g_i] + K[i]$$
$$a \leftarrow d,\quad d \leftarrow c,\quad c \leftarrow b,\quad b \leftarrow b + \text{ROTL}_{s_i}(\text{temp})$$

Where:
- $\text{AuxFunc}$ is one of $F, G, H, I$ depending on the round
- $M[g_i]$ accesses the message words in a round-specific scrambled order:
  - Round 1: $g_i = i$ (sequential access: 0, 1, 2, …, 15)
  - Round 2: $g_i = (5i + 1) \bmod 16$ (stride-5 access)
  - Round 3: $g_i = (3i + 5) \bmod 16$ (stride-3 access)
  - Round 4: $g_i = (7i) \bmod 16$ (stride-7 access)
- $K[i]$ is a per-operation constant: $K[i] = \lfloor 2^{32} \cdot |\sin(i+1)| \rfloor$ (uses the sine function to produce pseudo-random constants)
- $s_i$ is the left-rotation amount, cycling through $\{7, 12, 17, 22\}$ in round 1, $\{5, 9, 14, 20\}$ in round 2, $\{4, 11, 16, 23\}$ in round 3, $\{6, 10, 15, 21\}$ in round 4
- All additions are modulo $2^{32}$

4. **Update state** after all 64 operations:
$$A = A + a,\quad B = B + b,\quad C = C + c,\quad D = D + d \pmod{2^{32}}$$

This **Merkle-Damgård** construction: the output of one block feeds as the starting state for the next block.

<div align="center">
<!-- ![MD5 block processing diagram](./images/md5-block.png) -->
    <img src="./images/md5-block.png">
</div>

## Step 5: Produce the Output

After all blocks are processed, the hash is the 128-bit concatenation of the four state words in **little-endian** byte order:

$$\text{MD5}(m) = A \;\|\; B \;\|\; C \;\|\; D$$

## Weaknesses of MD5

MD5 is thoroughly broken:

- **1993**: Den Boer and Bosselaers found pseudo-collisions in MD5's compression function
- **1996**: Hans Dobbertin found collisions in the compression function itself
- **2004**: Wang et al. demonstrated full MD5 collisions in under one hour on a standard PC
- **2008**: Researchers created a rogue CA certificate by exploiting MD5 collisions in SSL/TLS
- **Today**: MD5 collisions can be found in **seconds** on commodity hardware

The collision attack works by finding two different 512-bit input blocks that produce the same output from the compression function. Because of the Merkle-Damgård structure, such a collision in one block propagates to a full-message collision.

> Never use MD5 for security-sensitive purposes: file integrity checks in security contexts, digital certificates, password hashing, or MACs. Use SHA-256 or SHA-3.

---

# 5. SHA-1 — Secure Hash Algorithm 1

## Overview

SHA-1 was published by NIST in 1995 (based on NSA's design) as a more secure successor to MD4/MD5. It dominated for over a decade in TLS certificates, GPG, SSH, and software signing.

**Parameters:**
- Output: **160 bits**
- Internal state: 160 bits (5 × 32-bit words)
- Block size: **512 bits** (16 × 32-bit big-endian words)
- Rounds: **80**
- Word size: 32 bits, **big-endian** (unlike MD5's little-endian)
- Max message: $2^{64} - 1$ bits

**Current status: broken.** SHA-1 collision resistance is fully compromised. Use SHA-256 or SHA-3.

## Initialization

SHA-1's five initial hash values:

$$h_0 = \texttt{0x67452301}$$
$$h_1 = \texttt{0xEFCDAB89}$$
$$h_2 = \texttt{0x98BADCFE}$$
$$h_3 = \texttt{0x10325476}$$
$$h_4 = \texttt{0xC3D2E1F0}$$

Note: $h_0$ through $h_3$ share their values with MD5's initialization constants, reflecting SHA-1's lineage from MD4.

## Padding

Identical structure to MD5, but with one critical difference: the 64-bit length is appended **big-endian** (most significant byte first), while MD5 uses little-endian.

## Message Schedule (Key Expansion)

SHA-1 expands the 16 × 32-bit words of each block into 80 words:

$$W[t] = M[t] \quad \text{for } 0 \leq t \leq 15$$
$$W[t] = \text{ROTL}_1(W[t-3] \oplus W[t-8] \oplus W[t-14] \oplus W[t-16]) \quad \text{for } 16 \leq t \leq 79$$

The rotation by 1 (absent in MD4) was specifically added by NSA to provide better avalanche and resist differential attacks. This expansion means every bit of the input block influences all 80 round inputs.

## The Four Round Functions

SHA-1 uses four functions over 80 steps, grouped into four ranges of 20 steps each:

| Steps | Name | Formula |
|---|---|---|
| 0–19 | Ch (Choose) | $(B \wedge C) \vee (\neg B \wedge D)$ |
| 20–39 | Parity | $B \oplus C \oplus D$ |
| 40–59 | Maj (Majority) | $(B \wedge C) \vee (B \wedge D) \vee (C \wedge D)$ |
| 60–79 | Parity | $B \oplus C \oplus D$ |

**Ch(B,C,D)**: "if B then C else D" — same as MD5's $F$, and the same as AES's S-box structure.

**Parity**: XOR of all three inputs — a lightweight mixer.

**Maj(B,C,D)**: outputs the majority bit — 1 if at least two of {B,C,D} are 1.

## Round Constants

Each group of 20 steps uses a constant derived from square roots of small integers:

| Steps | Constant $K_t$ | Value |
|---|---|---|
| 0–19 | $\lfloor \sqrt{2} \cdot 2^{30} \rfloor$ | `0x5A827999` |
| 20–39 | $\lfloor \sqrt{3} \cdot 2^{30} \rfloor$ | `0x6ED9EBA1` |
| 40–59 | $\lfloor \sqrt{5} \cdot 2^{30} \rfloor$ | `0x8F1BBCDC` |
| 60–79 | $\lfloor \sqrt{10} \cdot 2^{30} \rfloor$ | `0xCA62C1D6` |

## Processing Each Block

Initialize round variables: $A = h_0,\; B = h_1,\; C = h_2,\; D = h_3,\; E = h_4$

For $t = 0$ to $79$:
$$\text{TEMP} = \text{ROTL}_5(A) + f(t, B, C, D) + E + W[t] + K_t$$
$$E = D,\quad D = C,\quad C = \text{ROTL}_{30}(B),\quad B = A,\quad A = \text{TEMP}$$

After 80 steps, update:
$$h_0 \mathrel{+}= A,\quad h_1 \mathrel{+}= B,\quad h_2 \mathrel{+}= C,\quad h_3 \mathrel{+}= D,\quad h_4 \mathrel{+}= E \pmod{2^{32}}$$

## Output

$$\text{SHA-1}(m) = h_0 \;\|\; h_1 \;\|\; h_2 \;\|\; h_3 \;\|\; h_4$$

160 bits, big-endian.

<div align="center">
<!-- ![SHA-1 processing diagram](./images/sha1-block.png) -->
    <img src="./images/sha1-block.png">
</div>

## Weaknesses of SHA-1

- **2005**: Wang et al. reduced the attack cost to $2^{69}$, well below the $2^{80}$ birthday bound
- **2011**: Marc Stevens reduced further to $2^{61}$
- **2017**: Google's **SHAttered** attack produced the first known real SHA-1 collision — two different PDF files with identical SHA-1 hashes — costing approximately $2^{63.1}$ SHA-1 compressions (~110 GPU-years)
- **2020**: Leurent and Peyrin demonstrated chosen-prefix SHA-1 collisions, enabling attacks on PGP/GnuPG signatures

SHA-1 is fully deprecated by NIST. No new systems should use it.

---

# 6. SHA Algorithm Family — Comparison Table

The SHA family spans four generations. SHA-2 (SHA-256, SHA-512, and their truncated variants) is the current standard. SHA-3 (Keccak) is a completely different design, not based on Merkle-Damgård.

| Algorithm | Result (bit) | State (bit) | Block (bit) | Max Message (bit) | Cycles (Rounds) | Operations | Collision Security |
|---|:---:|:---:|:---:|:---:|:---:|---|:---:|
| **SHA-1** | 160 | 160 | 512 | $2^{64}-1$ | 80 | Ch, Parity, Maj, Parity (4 functions) | Broken ($\approx 2^{63}$) |
| **SHA-224** | 224 | 256 | 512 | $2^{64}-1$ | 64 | Ch, Maj, $\Sigma_0$, $\Sigma_1$, $\sigma_0$, $\sigma_1$ (6 functions) | $2^{112}$ |
| **SHA-256** | 256 | 256 | 512 | $2^{64}-1$ | 64 | Ch, Maj, $\Sigma_0$, $\Sigma_1$, $\sigma_0$, $\sigma_1$ (6 functions) | $2^{128}$ |
| **SHA-384** | 384 | 512 | 1024 | $2^{128}-1$ | 80 | Ch, Maj, $\Sigma_0$, $\Sigma_1$, $\sigma_0$, $\sigma_1$ (6 functions) | $2^{192}$ |
| **SHA-512** | 512 | 512 | 1024 | $2^{128}-1$ | 80 | Ch, Maj, $\Sigma_0$, $\Sigma_1$, $\sigma_0$, $\sigma_1$ (6 functions) | $2^{256}$ |
| **SHA-512/224** | 224 | 512 | 1024 | $2^{128}-1$ | 80 | Ch, Maj, $\Sigma_0$, $\Sigma_1$, $\sigma_0$, $\sigma_1$ (6 functions) | $2^{112}$ |
| **SHA-512/256** | 256 | 512 | 1024 | $2^{128}-1$ | 80 | Ch, Maj, $\Sigma_0$, $\Sigma_1$, $\sigma_0$, $\sigma_1$ (6 functions) | $2^{128}$ |
| **SHA3-224** | 224 | 1600 | 1152 | Unlimited | 24 | θ, ρ, π, χ, ι (Keccak-f[1600], 5 step mappings) | $2^{112}$ |
| **SHA3-256** | 256 | 1600 | 1088 | Unlimited | 24 | θ, ρ, π, χ, ι (Keccak-f[1600], 5 step mappings) | $2^{128}$ |
| **SHA3-384** | 384 | 1600 | 832 | Unlimited | 24 | θ, ρ, π, χ, ι (Keccak-f[1600], 5 step mappings) | $2^{192}$ |
| **SHA3-512** | 512 | 1600 | 576 | Unlimited | 24 | θ, ρ, π, χ, ι (Keccak-f[1600], 5 step mappings) | $2^{256}$ |

### Notes on the Table

**State vs Result**: For SHA-224, the state is 256 bits (same internal registers as SHA-256), but only 224 bits of the final state are output — the last 32 bits are truncated. Similarly, SHA-384 shares SHA-512's 512-bit state; SHA-512/224 and SHA-512/256 are SHA-512 with different initialization vectors and truncated output. This design reuses implementation infrastructure.

**Block size**: This is the size of one chunk of input consumed per compression function invocation. SHA-2 with 32-bit word size uses 512-bit blocks; SHA-2 with 64-bit word size uses 1024-bit blocks.

**SHA3 Block = Rate ($r$)**: SHA-3 uses a **sponge construction** (not Merkle-Damgård), where "block" means the number of bits absorbed per permutation call (the **rate** $r$). The state is always 1600 bits; the unused portion (the **capacity** $c = 1600 - r$) determines security. SHA3-256 has $r = 1088$, $c = 512$, giving $2^{256}$ collision resistance ($c/2 = 256$ bits).

**Cycles**: The compression/permutation is applied once per block/rate absorption. The number here is internal rounds inside one permutation invocation: 80 rounds for SHA-1, 64 for SHA-256, 80 for SHA-512, 24 for Keccak-f[1600].

**Operations (SHA-2 family)**: The six mixing functions used per step of the SHA-256/512 compression function are:
$$\text{Ch}(x,y,z) = (x \wedge y) \oplus (\neg x \wedge z)$$
$$\text{Maj}(x,y,z) = (x \wedge y) \oplus (x \wedge z) \oplus (y \wedge z)$$
$$\Sigma_0(x) = \text{ROTR}^a(x) \oplus \text{ROTR}^b(x) \oplus \text{ROTR}^c(x)$$
$$\Sigma_1(x) = \text{ROTR}^d(x) \oplus \text{ROTR}^e(x) \oplus \text{ROTR}^f(x)$$
$$\sigma_0(x) = \text{ROTR}^g(x) \oplus \text{ROTR}^h(x) \oplus \text{SHR}^i(x)$$
$$\sigma_1(x) = \text{ROTR}^j(x) \oplus \text{ROTR}^k(x) \oplus \text{SHR}^l(x)$$

(Rotation constants differ between SHA-256 and SHA-512 variants.)

**Operations (SHA-3/Keccak)**: The five step mappings of the Keccak-f permutation are:
- $\theta$: XOR each bit with the parity of two adjacent columns
- $\rho$: rotate each of the 25 lanes by a fixed offset
- $\pi$: rearrange the 25 lanes into a new pattern
- $\chi$: nonlinear layer — the only nonlinear step ($a \oplus (\neg b \wedge c)$ per lane)
- $\iota$: XOR a round constant into one lane to break symmetry

---

# 7. The Merkle-Damgård Construction

All of MD5, SHA-1, and SHA-2 are built on the **Merkle-Damgård (MD) construction** — the same underlying framework.

The idea:

1. Pad the message and split it into fixed-size blocks $M_1, M_2, \ldots, M_t$
2. Start with a fixed **initial value (IV)**: $H_0 = \text{IV}$
3. For each block: $H_i = f(H_{i-1},\; M_i)$ — apply a **compression function** $f$ that takes the previous hash state and the current message block and produces the next state
4. The final state $H_t$ is the digest

$$\text{IV} \xrightarrow{M_1} H_1 \xrightarrow{M_2} H_2 \;\cdots\; \xrightarrow{M_t} H_t = \text{digest}$$

<div align="center">
<!-- ![Merkle-Damgård construction](./images/merkle-damgard.png) -->
    <img src="./images/merkle-damgard.png">
</div>

**Security**: Merkle and Damgård proved that if the compression function $f$ is collision-resistant, the full hash function is also collision-resistant. So the security of MD5/SHA reduces to the security of their respective compression functions.

**Weakness: Length Extension Attack**

Because the hash state after processing all blocks is the final output, an attacker who knows $H(m)$ and the length of $m$ can compute $H(m \| \text{padding} \| m')$ for any suffix $m'$ — **without knowing $m$**. They simply initialize a new MD computation starting at $H(m)$ and feed $m'$.

This breaks naive constructions like $\text{MAC} = H(K \| m)$ — the attacker can extend $m$ without knowing $K$. This is why HMAC uses the double-hash construction.

SHA-3 (sponge construction) is immune to length extension attacks by design.

---

# 8. SHA-3 and the Sponge Construction

SHA-3 was selected in 2012 through a NIST competition, as a backup standard distinct in design from SHA-2. Its underlying function is **Keccak** (designed by Bertoni, Daemen, Peeters, Van Assche).

## The Sponge Construction

Instead of Merkle-Damgård's linear chaining, SHA-3 uses a **sponge**:

$$\text{State} = b \text{ bits} = r + c \text{ bits}$$

where $r$ = **rate** (bits absorbed/squeezed per step) and $c$ = **capacity** (security parameter).

**Absorb phase**: for each $r$-bit block of padded input:
1. XOR the block into the first $r$ bits of the state
2. Apply the Keccak-f permutation to the full $b$-bit state

**Squeeze phase**: output the first $r$ bits of the state (repeat with more permutation applications if more output is needed).

The capacity $c = 1600 - r$ never touches the input directly. Its size governs security: collision resistance = $c/2$ bits.

This design is naturally immune to length-extension attacks, supports variable output lengths (SHAKE-128, SHAKE-256), and uses a permutation rather than a compression function.

---

# 9. Security Summary

| Function | Output | Collision | Pre-image | 2nd Pre-image | Status |
|---|:---:|:---:|:---:|:---:|---|
| MD5 | 128 | **Broken** ($< 1$s) | $2^{128}$ (theoretical) | $2^{128}$ (theoretical) | **Retired** |
| SHA-1 | 160 | **Broken** ($2^{63}$) | $2^{160}$ (theoretical) | $2^{160}$ (theoretical) | **Deprecated** |
| SHA-256 | 256 | $2^{128}$ | $2^{256}$ | $2^{256}$ | **Secure** |
| SHA-512 | 512 | $2^{256}$ | $2^{512}$ | $2^{512}$ | **Secure** |
| SHA3-256 | 256 | $2^{128}$ | $2^{256}$ | $2^{256}$ | **Secure** |
| SHA3-512 | 512 | $2^{256}$ | $2^{512}$ | $2^{512}$ | **Secure** |

**Practical guidance:**
- Password hashing: use **bcrypt**, **scrypt**, or **Argon2** (purpose-built to be slow and memory-hard — SHA-256 is too fast)
- Data integrity, digital signatures: **SHA-256** or **SHA-512**
- MACs: **HMAC-SHA-256** or **HMAC-SHA-512**
- Post-quantum context: SHA-3 variants are preferred as they rely on a sponge permutation with no known quantum speedup beyond Grover's $2^{n/2}$
