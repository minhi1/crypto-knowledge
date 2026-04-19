# 1. The Core Philosophy

Both DES and AES belong to a family called **block ciphers**. Instead of encrypting data one bit at a time (like a stream cipher), they take a fixed-size block of plaintext and transform it into a block of ciphertext.

Historically, simple encryption relied on either substitution (replacing 'A' with 'D') or transposition (shuffling the order of letters). However, using only substitution or transposition is fundamentally unsafe due to the inherent statistical nature of human language.

To solve this, DES and AES are built as product ciphers. 

>They loop data through multiple consecutive cycles—called rounds—where each round applies a combination of substitutions and transpositions. In each round, a specific sub-key (a "round key") generated from your master secret key is injected into the math.

This looping structure is designed to achieve Claude Shannon's two primary goals for a secure cipher:
- **Confusion**: Making the mathematical relationship between the key and the ciphertext as complex and non-linear as possible. If a hacker analyzes the ciphertext, they shouldn't be able to deduce anything about the key.
- **Diffusion**: Spreading the statistical structure of the plaintext across the entire ciphertext.

## Challenge

Imagine we are encrypting a 64-bit block of data with a secure block cipher. Then, we encrypt that exact same block again using the exact same key, but this time we flip just one single bit of the plaintext (from 0 to 1).

<u>Based on the concept of diffusion, what should ideally happen to the resulting ciphertext compared to the first one?</u>

Imagine we have a cup of clear water. Then add a tiny drop of red dye and stir it vigorously. The color will gradually diffuse and turn the entire glass of water pink.

Similarly, in block cipher, the stirring action is equivalent to multiple rounds of substitutions and transpositions. Claude Shannon's goal for diffusion was to ensure that the mathematical structure of the input spreads everywhere in the output. 

As a result, the flipped-bit cipher is much different from the first one. The one flipped bit enters the first round of encryption, changes a few bits. Those few bits enter the second round and change more.

# 2. The DES Architecture

DES (Data Encryption Standard) operates as a symmetric (same key for encryption - decryption between sender and receiver) key block cipher, processing data in fixed-side blocks of 64 bits using a 56-bit key.

The encryption process consists of several consecutive encryption rounds. For DES, this happens 16 times.

The genius of DES lies in its internal structure, known as a **Feistel Network**. A Feistel Network takes the 64-bit block and chops it exactly in half: both left-half and right-half are 32 bits.

## The Feistel Round

1. **The Mangler Function** (*Mixing data and key*): The right half is sent into a complex mathematical function (usually denoted as $f$) alongside a secret sub-key for that specific round. This scrambles the data based on the key.
2. **The XOR Mix** (*Combining the halves*): The output from Mangler Function is then mathematically merged with the left half using XOR.
3. **The Swap** (*Setting up the next round*): The two halves trade places. Then the original right half (which remains untouched) becomes the new left half.

![Feistel Round](./images/image.png)

## Challenge

The mangler function $f$ is mathematically irreversible. Once the right half and the sub-key go through it, we cannot reverse-engineer the math to get the original data back.

But the receiver must be able to decrypt and read it.

<u>How can the receiver manages to decrypt the ciphertext back into plaintext?</u> (*Hint: Think about the Left/Right swap and the keys used in those 16 rounds*)

The receiver doesn't need to reverse the math of $f$ function. They simply run the ciphertext in the same 16 rounds, but this time, **use the keys in reversed order**. The XOR operation will cancle itself out, leads to unlocking the data. Magic.

## Initial Permutation (IP) and Final Permutation (FP)

Before the data even hits the Feistel network, the 64-bit plaintext goes through the IP (Initial Permutation). This simply shuffles the order of the bits according to a fixed table. After the 16 encryption rounds are finished, the FP (Final Permutation) un-shuffles them.

>FP and IP have no cryptographic significance, only for loading data into and out of data blocks (in the mid-1970s hardware mechanism)

>$FP=IP^{-1}$

**Initial Permutation Table**

![Initial Permutation Table](./images/image-1.png)​

**Final Permutation Table**

![Final Permutation Table](./images/image-2.png)

## Key Expansion and Key Schedule

**Key Expansion:** It is the way through which we get 16 subkeys of 48 bits from the initial 64 bit key for each round of DES. The generated keys will be used during the encryption of plaintext.


We start with a 64-bit key, but 8 bits are used for parity (error checking).

The key size reduces from 64 to 56 bits via permutation box PC-1.

![PC-1](./images/image-3.png)

The first entry of the table is 57, means that the 57th bit of the original key $K$ becomes the first bit of the permuted key.

Next, we split the permuted keys (56 bits) into left and right halves $C0$, $D0$ of 28 bits each. Now we get 16 blocks of $Cn$, $Dn$ ($1 \leq n \leq 16$) by applying the cyclic left shifts (starting from $C0$,$D0$) based on the following rules:
- With subkey in {1, 2, 9, 16}: rotate to the left 1 position.
- With remaining: rotate to the left 2 positions.

> The rotation of $Ci$, $Di$ is based on $C_{i-1}$ and $D_{i-1}$.

Now for each pair of $Ci$, $Di$, combine them into $CiDi$ and apply permutation PC-2 to reduce from 56 bits to 48 bits.

![PC-2](./images/image-4.png)

Finally, we have 16 sub-keys to schedule for each round.

## Inside the Mangler Function (Function $f$ of DES)

When the right half (32-bit) enters the $f$ function, it goes through 4 stages:
1. **The Expansion (E-Box):** Here, we will XOR the 32-bit data and the sub-key (48-bit). Since they don't match in size, the E-Box takes the 32-bit and expands them to 48 bits.
2.  **The Mix:** The 48-bit expanded data is XOR with the round key.
3. **The S-boxes (Substitution):** The 48 bits are chopped into eight separate 6-bit chunks. Each chunk goes into its own specific S-box (from S1 to S8). Each S-box is a lookup table that receives 6-bit input and output 4 bits. So that 8 boxes produce 32 bits to XOR against the left half of original data. The lookup process for S-box is as follow:
    - Suppose the input bits is *abcdef*.
    - The row to lookup would be calculated by combining first and last bit: *af*
    - The column to lookup is combination of remaining bits: *bcde*
4. **The P-box (Permutation)**: Finally, those resulting 32-bits are shuffled one last time before exiting the $f$ function to be XOR against the left half of original data.

**E-Box**

![E-Box](./images/image-5.png)

**S-boxes**

![S-Box 1 and 2](./images/image-6.png)

![S-Box 3 and 4](./images/image-7.png)

![S-Box 5 and 6](./images/image-8.png)

![S-Box 7 and 8](./images/image-9.png)

**P-box**

![P-box](./images/image-10.png)

## Weakness of DES

DES had one fatal flaw: **The key was too short**.

With strong computers nowadays, the short 56-bit key is vulnerable to brute-force attack.