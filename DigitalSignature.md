# 1. What is a Digital Signature?

Encryption solves confidentiality: only the intended recipient can read the message. But it solves nothing else. An attacker could intercept an encrypted message, modify a few ciphertext bytes, and forward it — the recipient decrypts something garbled with no idea it was tampered with. And even if the message is intact, how does the recipient know it actually came from who they think it did?

A **digital signature** solves three problems at once:

- **Authentication**: confirms the message was produced by a specific party who holds a particular private key
- **Integrity**: any modification to the signed message — even one flipped bit — invalidates the signature
- **Non-repudiation**: the signer cannot later deny having signed; the signature is mathematical proof tied to their unique private key

## The General Model

A digital signature scheme has three algorithms:

1. **KeyGen** → $(sk, vk)$: generates a secret signing key $sk$ and a public verification key $vk$
2. **Sign$(sk, m)$** → $\sigma$: produces a signature $\sigma$ on message $m$ using $sk$
3. **Verify$(vk, m, \sigma)$** → accept/reject: checks whether $\sigma$ is a valid signature on $m$ under $vk$

Correctness: for all messages $m$, Verify$(vk, m,$ Sign$(sk, m))$ = accept.

Security: an attacker who only knows $vk$ (the public key) cannot produce a valid $(m, \sigma)$ pair for any new message $m$ — even after seeing valid signatures on other messages.

## Why Hash Before Signing?

Most real-world signature schemes do not sign the message $m$ directly. They sign $H(m)$, the output of a hash function like SHA-256. This is important for three reasons:

1. **Size**: signature algorithms operate on fixed-size values (e.g., 256 bytes for RSA-2048). A 10 MB file cannot be exponentiated directly.
2. **Security**: signing raw messages exposes structural algebraic properties that can be exploited (more on this later). Hashing destroys that structure.
3. **Efficiency**: hashing a large file is fast; you only perform the expensive cryptographic operation on a small fixed-size output.

> Throughout this document: when we write "sign message $m$", we mean sign the hash $H(m)$. Only textbook/toy examples operate on $m$ directly.

---

# 2. RSA Signature

## The Scheme

RSA signature runs RSA "in reverse" compared to RSA encryption. The signer uses their **private key** to sign; anyone with the **public key** can verify.

**Key generation**: same as RSA encryption — choose primes $p$, $q$, compute $n = pq$, $\phi(n) = (p-1)(q-1)$, pick $e$ with $\gcd(e, \phi(n)) = 1$, compute $d \equiv e^{-1} \pmod{\phi(n)}$.

- Public verification key: $(e,\; n)$
- Secret signing key: $(d,\; n)$

### Signing

Compute the hash of the message: $h = H(m)$

Compute the signature:
$$\sigma = h^d \bmod n$$

Publish $(m,\; \sigma)$.

### Verification

Given $(m, \sigma)$ and public key $(e, n)$:

1. Recompute the hash: $h = H(m)$
2. Compute: $h' = \sigma^e \bmod n$
3. Accept iff $h' = h$

### Why Verification Works

$$\sigma^e = (h^d)^e = h^{de} \pmod{n}$$

Since $de \equiv 1 \pmod{\phi(n)}$, we have $de = 1 + k\phi(n)$, and by Euler's theorem:

$$h^{de} = h^{1 + k\phi(n)} = h \cdot (h^{\phi(n)})^k \equiv h \cdot 1 = h \pmod{n}$$

So $h' = h$, and the verification check passes.

## Full Worked Example

**Setup:** $p = 5$, $q = 11$, $n = 55$, $\phi(n) = 40$, $e = 3$, $d = 27$.

Let the "hash" of the message be $h = 2$ (simplified for arithmetic).

**Signing:**
$$\sigma = 2^{27} \bmod 55$$

Compute by repeated squaring ($27 = 16 + 8 + 2 + 1$):

| Power | Value mod 55 |
|---|---|
| $2^1$ | 2 |
| $2^2$ | 4 |
| $2^4$ | 16 |
| $2^8$ | $16^2 = 256 \bmod 55 = 256 - 4{\times}55 = 36$ |
| $2^{16}$ | $36^2 = 1296 \bmod 55 = 1296 - 23{\times}55 = 31$ |

$\sigma = 31 \times 36 \times 4 \times 2 \bmod 55$

$= 31 \times 36 \bmod 55 = 1116 \bmod 55 = 1116 - 20{\times}55 = 16$

$= 16 \times 4 \bmod 55 = 64 \bmod 55 = 9$

$= 9 \times 2 \bmod 55 = 18$

So $\sigma = 18$.

**Verification:**
$$\sigma^e \bmod n = 18^3 \bmod 55 = 5832 \bmod 55 = 5832 - 106{\times}55 = 5832 - 5830 = 2$$

$h' = 2 = h$ ✓ — signature accepted.

---

## Attacks on RSA Signatures

### Attack 1 — Factorizing $n$

This is the root attack. If an attacker can factor $n = pq$, the entire private key collapses:

1. Factor $n$ → recover $p$ and $q$
2. Compute $\phi(n) = (p-1)(q-1)$
3. Run the Extended Euclidean Algorithm: $d = e^{-1} \bmod \phi(n)$
4. With $d$ in hand, the attacker can forge a valid signature $\sigma = H(m)^d \bmod n$ on any message $m$ of their choosing

This attack is computationally infeasible for well-chosen 2048-bit $n$ using the best known algorithms (GNFS). However, RSA moduli with special structure (e.g., primes too close together, or one prime sharing a factor with another key's modulus) have been factored in practice.

**Defense**: use 2048-bit or larger keys; generate $p$ and $q$ independently and randomly; verify $|p - q|$ is large.

---

### Attack 2 — Multiple Key Pairs Yielding the Same Signature

This attack has two related faces.

#### Face A: Multiple Valid Private Keys (Carmichael's Function)

RSA key generation computes $d = e^{-1} \bmod \phi(n)$, where $\phi(n) = (p-1)(q-1)$. However, the *actual* mathematical requirement for RSA to work is weaker: we only need $ed \equiv 1 \pmod{\lambda(n)}$, where $\lambda(n)$ is the **Carmichael function**:

$$\lambda(n) = \lambda(pq) = \text{lcm}(p-1,\; q-1)$$

Since $\lambda(n)$ divides $\phi(n)$, we have $\phi(n) = k \cdot \lambda(n)$ for some integer $k \geq 1$.

This means there are *multiple valid private exponents* for the same public key $(e, n)$:

$$d_0 = e^{-1} \bmod \lambda(n)$$
$$d_1 = d_0 + \lambda(n), \quad d_2 = d_0 + 2\lambda(n), \quad \ldots$$

All of $d_0, d_1, d_2, \ldots$ produce valid signatures that verify correctly under the same $(e, n)$.

**Implication:** If an attacker steals one valid private key $d_i$, all other equivalent keys follow trivially. More importantly, if $k = \phi(n)/\lambda(n)$ is large (which happens when $\gcd(p-1, q-1)$ is large), an attacker who searches for keys of the form $e^{-1} \bmod \lambda(n)$ might find a shorter $d_0$ — and use it in Wiener's attack (small $d$ attack).

**Defense**: choose $p$ and $q$ such that both $(p-1)$ and $(q-1)$ have large prime factors (strong primes). This maximizes $\lambda(n)$ and minimizes the number of equivalent keys.

#### Face B: Existential Forgery (No-Message Attack)

Even without knowing $d$, an attacker can produce a valid (message, signature) pair using textbook RSA:

1. Choose any arbitrary value $\sigma$ as the "signature"
2. Compute $m = \sigma^e \bmod n$
3. $(m,\; \sigma)$ is a valid signature pair — verification succeeds: $\sigma^e \equiv m \pmod{n}$ ✓

The attack works because the attacker chose $\sigma$ first and back-computed $m$. They cannot choose a *meaningful* $m$ — the resulting $m$ will look like random noise.

**Defense**: apply a hash function before signing. The attacker cannot reverse-engineer a meaningful document whose SHA-256 hash equals a chosen $m$. With $\sigma = H(m)^d \bmod n$, the forged $m$ from $\sigma^e$ would need to equal $H(\text{some document})$ — which is computationally infeasible (preimage resistance).

---

### Attack 3 — The Homomorphic Property

RSA has a **multiplicative homomorphism**: for any two values $a$ and $b$,

$$(a \cdot b)^d \equiv a^d \cdot b^d \pmod{n}$$

This means: a valid signature on the *product* of two messages can be computed by *multiplying* the signatures on each message individually.

**Concrete attack scenario:**

Alice holds private key $d$. She has legitimately signed two messages:
- $\sigma_1 = m_1^d \bmod n$ (signature on $m_1$)
- $\sigma_2 = m_2^d \bmod n$ (signature on $m_2$)

An attacker observes both signatures and computes:
$$\sigma^* = \sigma_1 \cdot \sigma_2 \bmod n = m_1^d \cdot m_2^d = (m_1 \cdot m_2)^d \bmod n$$

Now $\sigma^*$ is a valid RSA signature on $m^* = m_1 \cdot m_2 \bmod n$ — and Alice never signed $m^*$.

If $m_1 \cdot m_2 \bmod n$ happens to be a meaningful value (e.g., another document, or a hash value in a weak scheme), the attacker has forged a signature with zero knowledge of $d$.

**Defense**: sign the hash $H(m)$, not $m$ directly. Since $H(m_1 \cdot m_2) \neq H(m_1) \cdot H(m_2)$ for any collision-resistant hash, the homomorphic property of RSA cannot propagate through the hash function.

## Challenge

Alice signs a document authorizing "Transfer: \$100 to Bob" by computing $\sigma = H(\text{doc})^d \bmod n$ and publishes $(\text{doc}, \sigma)$. Later she also signs an unrelated document producing $\sigma_2$.

<u>Even with hash-based RSA, what must the attacker still worry about when trying to use the homomorphic attack?</u>

The attacker computes $\sigma_1 \cdot \sigma_2 \bmod n$ and gets a valid RSA signature on $H(\text{doc}_1) \cdot H(\text{doc}_2) \bmod n$. But this value is not $H(\text{doc}_1 \cdot \text{doc}_2)$ — it's the *product of two hash outputs*, which is itself not a hash of anything meaningful. The forged "signature" verifies as a signature on the value $H(\text{doc}_1) \cdot H(\text{doc}_2) \bmod n$, but that value isn't the hash of any document an attacker would want to forge. The hash function acts as a firewall: the homomorphism applies to the algebraic layer (RSA), but the semantic layer (documents) is protected by hash preimage resistance.

---

# 3. ElGamal Signature

## Foundation: The Discrete Logarithm Problem

ElGamal's security rests on a completely different hard problem than RSA. Instead of factoring, it relies on the **Discrete Logarithm Problem (DLP)**:

> Given a large prime $p$, a generator $g$ of $\mathbb{Z}_p^*$, and the value $y = g^x \bmod p$ — find $x$.

A **generator** $g$ of $\mathbb{Z}_p^*$ is a value whose powers cycle through all of $\{1, 2, \ldots, p-1\}$ before repeating. Computing $y$ from $x$ is fast (repeated squaring). Going backwards — finding $x$ from $y$ — has no known polynomial-time algorithm for large $p$. This asymmetry is the trapdoor.

## Key Generation

Choose a large prime $p$ and a generator $g$ of $\mathbb{Z}_p^*$. These can be public system parameters shared by all users.

1. Choose a random private key: $x$, where $1 \leq x \leq p-2$
2. Compute the public key: $y = g^x \bmod p$

- **Public verification key**: $(p,\; g,\; y)$
- **Secret signing key**: $x$

## Signing

Let $h = H(m)$ be the hash of the message.

1. Choose a **random ephemeral key** $k$, where $1 \leq k \leq p-2$ and $\gcd(k,\; p-1) = 1$
2. Compute:

$$r = g^k \bmod p$$

$$s = k^{-1}(h - x \cdot r) \bmod (p-1)$$

3. If $s = 0$, choose a new $k$ and repeat
4. The signature is $(\,r,\; s\,)$

> $k$ must be chosen **fresh and uniformly at random** for every single signature. Reusing $k$ even once is catastrophic (see below).

## Verification

Given message $m$, signature $(r, s)$, and public key $(p, g, y)$:

1. Check validity: $0 < r < p$ and $0 < s < p-1$. Reject if not.
2. Compute $h = H(m)$
3. Check whether:

$$g^h \equiv y^r \cdot r^s \pmod{p}$$

Accept if the equation holds; reject otherwise.

## Why Verification Works

We need to show that the verification equation holds for a legitimately generated signature.

From the signing step: $s = k^{-1}(h - xr) \bmod (p-1)$

Multiply both sides by $k$: $\;\;ks \equiv h - xr \pmod{p-1}$

Rearrange: $\;\;h \equiv xr + ks \pmod{p-1}$

Now use Fermat's Little Theorem ($a^{p-1} \equiv 1 \pmod{p}$ for prime $p$) to compute:

$$g^h \equiv g^{xr + ks} = g^{xr} \cdot g^{ks} = (g^x)^r \cdot (g^k)^s = y^r \cdot r^s \pmod{p}$$

The verification equation is exactly this identity. ✓

## The $k$-Reuse Attack

This is the most devastating practical attack on ElGamal (and DSA). If the same $k$ is used to sign two different messages $m_1$ and $m_2$:

$$s_1 = k^{-1}(h_1 - xr) \bmod (p-1)$$
$$s_2 = k^{-1}(h_2 - xr) \bmod (p-1)$$

Subtract the two equations:

$$s_1 - s_2 \equiv k^{-1}(h_1 - h_2) \pmod{p-1}$$

The attacker knows $h_1$, $h_2$, $s_1$, $s_2$ (all public). They solve for $k$:

$$k \equiv (h_1 - h_2)(s_1 - s_2)^{-1} \pmod{p-1}$$

And then immediately recover the private key $x$:

$$x \equiv (h_1 - k \cdot s_1) \cdot r^{-1} \pmod{p-1}$$

**Every signature ever produced with key $x$ is now forgeable.**

This is not a theoretical attack. In 2010, Sony's PlayStation 3 was broken this way — they used a **fixed constant** instead of a random $k$ when signing firmware. Hackers extracted the private key and could sign any firmware image, enabling homebrew and piracy.

## Challenge

An attacker intercepts two ElGamal signatures from Alice: $(r_1, s_1)$ on $m_1$ and $(r_2, s_2)$ on $m_2$, and notices $r_1 = r_2$.

<u>What does $r_1 = r_2$ immediately tell the attacker, and what can they do next?</u>

$r = g^k \bmod p$. Two equal $r$ values mean the **same $k$ was used** for both signatures. The attacker can then compute $k = (h_1 - h_2)(s_1 - s_2)^{-1} \bmod (p-1)$ and recover Alice's private key $x$ directly, allowing them to forge any signature on any message — all because $k$ was reused once.

---

# 4. DSA — Digital Signature Algorithm

DSA is the U.S. federal standard (FIPS 186), designed by NIST and derived from ElGamal. It was engineered specifically for signing (not encryption) and produces shorter signatures.

## Why DSA Instead of ElGamal?

Raw ElGamal signatures on a 1024-bit prime $p$ are $2 \times 1024 = 2048$ bits. DSA shrinks signatures to $2 \times 160 = 320$ bits by introducing a second prime $q$ that defines a much smaller subgroup.

## Domain Parameters (Public, Shared by All Users)

1. **$p$**: a large prime, typically 1024–3072 bits
2. **$q$**: a prime divisor of $p - 1$, typically 160–256 bits. This means $q \mid (p-1)$.
3. **$g$**: a generator of the unique subgroup of $\mathbb{Z}_p^*$ of order $q$, computed as:

$$g = h^{(p-1)/q} \bmod p \quad \text{for any } h \text{ with } g \neq 1$$

The subgroup generated by $g$ has exactly $q$ elements: $\{g^0, g^1, \ldots, g^{q-1}\} \bmod p$.

## Key Generation

1. Choose random private key $x$: $\quad 1 \leq x \leq q-1$
2. Compute public key: $\quad y = g^x \bmod p$

- **Public verification key**: $(p,\; q,\; g,\; y)$
- **Secret signing key**: $x$

## Signing

Let $h = H(m)$, truncated to the bit-length of $q$ if necessary.

1. Choose random $k$: $\quad 1 \leq k \leq q-1$ (fresh and uniform every time)
2. Compute:

$$r = (g^k \bmod p) \bmod q$$

$$s = k^{-1}(h + x \cdot r) \bmod q$$

3. If $r = 0$ or $s = 0$, choose a new $k$ and repeat
4. Signature: $(\,r,\; s\,)$

Note both $r$ and $s$ are reduced modulo $q$ — that's where the short signature comes from.

## Verification

Given $m$, signature $(r, s)$, public key $(p, q, g, y)$:

1. Check: $0 < r < q$ and $0 < s < q$. Reject if not.
2. Compute $h = H(m)$
3. Compute:

$$w = s^{-1} \bmod q$$

$$u_1 = h \cdot w \bmod q$$

$$u_2 = r \cdot w \bmod q$$

$$v = (g^{u_1} \cdot y^{u_2} \bmod p) \bmod q$$

4. Accept iff $v = r$

## Why Verification Works

From the signing step: $s = k^{-1}(h + xr) \bmod q$

Multiply both sides by $k s^{-1}$: $\quad k \equiv (h + xr) \cdot s^{-1} = h w + xr w = u_1 + x \cdot u_2 \pmod{q}$

Now compute:

$$g^{u_1} \cdot y^{u_2} = g^{u_1} \cdot (g^x)^{u_2} = g^{u_1 + x \cdot u_2} = g^k \pmod{p}$$

So $v = (g^k \bmod p) \bmod q = r$ ✓

## Comparison: DSA vs ElGamal

| Property | ElGamal | DSA |
|---|---|---|
| Signature size | $2 \times |p|$ bits | $2 \times |q|$ bits (much shorter) |
| Security basis | DLP in $\mathbb{Z}_p^*$ | DLP in subgroup of order $q$ |
| Standardized | No | Yes (FIPS 186) |
| $k$-reuse vulnerability | Yes — catastrophic | Yes — equally catastrophic |
| Can encrypt? | Yes | No (signatures only) |

DSA carries the same fatal weakness as ElGamal: **$k$ must never be reused**. The recovery formula is identical — given two signatures with the same $k$, the private key $x$ is fully exposed.

> Modern practice replaces DSA with **ECDSA** (Elliptic Curve DSA) or **EdDSA** (Edwards-curve DSA). ECDSA uses the same math but in the smaller, more efficient group of an elliptic curve. EdDSA (e.g., Ed25519) uses a deterministic $k$ derived from the private key and the message, eliminating the $k$-reuse risk entirely.

---

# 5. One-Time Signature — The Rabin/Lamport Scheme

All the signature schemes above (RSA, ElGamal, DSA) require number-theoretic hardness assumptions: factoring, or discrete logarithm. A one-time signature scheme achieves security from a much simpler and more primitive assumption: **preimage resistance of a hash function**.

## Core Idea

Instead of building security on algebraic structure, we hide secret values behind a one-way function. The signer reveals the right secret values to match a message's bits; the verifier checks them against pre-published hash commitments.

The catch: the secret values are consumed by signing. Reveal enough of them to sign one message, and an attacker can potentially mix and match them to sign a different message. This is why the scheme is **one-time**: each key pair can safely sign exactly one message.

## Key Generation

Choose a security parameter $n$ equal to the bit-length of the hash output (e.g., $n = 256$ for SHA-256).

For each bit position $i$ from $1$ to $n$:
- Generate two random secret values: $x_i^0$ and $x_i^1$
- Compute their hashes: $y_i^0 = H(x_i^0)$ and $y_i^1 = H(x_i^1)$

$$\text{Secret signing key:} \quad SK = \{(x_i^0,\; x_i^1)\}_{i=1}^{n}$$

$$\text{Public verification key:} \quad VK = \{(y_i^0,\; y_i^1)\}_{i=1}^{n}$$

The public key contains $2n$ hash values. For $n = 256$, that's 512 hash values — a large key, but this is the price of simplicity.

## Signing

Compute the message hash: $\mathbf{h} = H(m) = h_1\, h_2\, \ldots\, h_n$ (a string of $n$ bits).

For each bit position $i$, reveal the secret value corresponding to bit $h_i$:

$$\sigma_i = x_i^{h_i}$$

The signature is: $\sigma = (\sigma_1,\; \sigma_2,\; \ldots,\; \sigma_n)$

In plain English: for each bit of the hash, you reveal the "0-secret" if the bit is 0, or the "1-secret" if the bit is 1. You always reveal exactly half of your secret values — one from each pair.

## Verification

Given message $m$, signature $\sigma = (\sigma_1, \ldots, \sigma_n)$, and public key $VK$:

1. Compute $\mathbf{h} = H(m) = h_1\, h_2\, \ldots\, h_n$
2. For each $i = 1$ to $n$: check that $H(\sigma_i) = y_i^{h_i}$
3. Accept iff all $n$ checks pass

The verifier hashes each revealed secret and checks it against the pre-committed public hash.

## Why It's Secure

Security reduces entirely to **preimage resistance** of $H$. An attacker who wants to forge a signature on a different message $m'$ needs to produce $\sigma_i' = x_i^{h_i'}$ for each bit $h_i'$ of $H(m')$.

For every bit position $i$ where $h_i' \neq h_i$ (i.e., the hash of $m'$ differs from the hash of $m$), the attacker needs to produce the *other* secret — the one the signer never revealed. That secret is only known from $y_i^{1-h_i} = H(x_i^{1-h_i})$. Inverting a hash to find the preimage is computationally infeasible.

## Why It's One-Time: The Reuse Attack

If the signer uses the same key to sign a second message $m'$ with $H(m') \neq H(m)$, the scheme breaks.

Suppose $H(m)$ and $H(m')$ differ at position $i$: $h_i = 0$ and $h_i' = 1$.

- Signing $m$ reveals: $x_i^0$ (the 0-secret at position $i$)
- Signing $m'$ reveals: $x_i^1$ (the 1-secret at position $i$)

After seeing both signatures, the attacker has **both** $x_i^0$ and $x_i^1$ for every position $i$ where the hashes differed. Once they collect both secrets for each position (across multiple signatures from the same key), they can sign *any* message by selecting whichever secret they need for each bit.

**In the worst case**: if you sign just two messages whose hashes together cover all $n$ bit values — e.g., one message where every hash bit is 0, and one where every bit is 1 — the attacker has all $2n$ secrets and can forge any signature.

## Practical Limitations and Extensions

| Property | Value |
|---|---|
| Security assumption | Hash preimage resistance only |
| Key size | $2n$ hash values (large) |
| Signature size | $n$ hash preimages (also large) |
| Messages per key | **Exactly 1** |
| Computation | Only hash evaluations — no modular exponentiation |
| Post-quantum secure | Yes — no quantum speedup for hash preimage inversion |x

The one-time restriction is a severe limitation. In practice, the scheme is extended using a **Merkle hash tree**:

- Generate a large number of one-time key pairs (e.g., $2^{20}$)
- Arrange their public verification keys as leaves of a binary tree
- Hash up the tree to a single root value — the **master public key**
- Each signature includes a tree authentication path proving the one-time key is in the tree
- Each leaf is used exactly once; the signer tracks which leaves remain unused

This construction — the **Merkle signature scheme** — is a **post-quantum secure** digital signature: its security does not rely on factoring or discrete logarithm, both of which are broken by Shor's algorithm on a quantum computer. Only hash preimage resistance is needed, and no quantum algorithm attacks that significantly.

## Challenge

<u>Why can a one-time signature scheme get away with needing only hash preimage resistance, while RSA and ElGamal need much stronger assumptions (factoring and DLP)?</u>

In RSA and ElGamal, the signer exposes the same public key repeatedly, so an attacker can observe many signatures and mount algebraic attacks — attempting to solve the underlying hard problem. The signature equation itself is algebraic and can be analyzed. In a one-time scheme, the signer literally reveals the secret values directly — there is no algebraic structure to attack. Security reduces purely to "you cannot find a value that hashes to a given output", which is a much simpler and more primitive guarantee. The price is that the key is consumed upon use: once you've revealed values, the algebraic-free guarantee is spent.

---

# 6. Summary: Digital Signature Schemes

| Scheme | Hardness Assumption | Key Size | Signature Size | One-Time? | $k$-Reuse Risk |
|---|---|---|---|:---:|:---:|
| RSA Signature | Integer factorization | $O(|n|)$ | $O(|n|)$ | No | N/A |
| ElGamal | Discrete logarithm | $O(|p|)$ | $2 \times O(|p|)$ | No | Yes — catastrophic |
| DSA | DLP in subgroup | $O(|p|)$ | $2 \times O(|q|)$ | No | Yes — catastrophic |
| Rabin/Lamport OTS | Hash preimage | $2n$ hashes | $n$ preimages | **Yes** | N/A |

**Choosing a scheme in practice (2025):**
- For general-purpose signatures: **Ed25519** (EdDSA on Curve25519) — fast, deterministic nonce, compact
- For post-quantum security: **CRYSTALS-Dilithium** or **SPHINCS+** (hash-based, standardized by NIST in 2024)
- Avoid: textbook RSA without padding, ElGamal/DSA with any randomness weakness, RSA-1024
