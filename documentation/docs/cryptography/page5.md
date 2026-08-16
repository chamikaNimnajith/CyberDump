# Data Encryption Standard (DES)

## 1. Introduction to DES

**Data Encryption Standard (DES)** is a historical **symmetric-key block cipher** that played an important role in the development of modern cryptography.

DES uses the **same secret key** for both encryption and decryption.

```text
Plaintext
    │
    │  56-bit effective key
    ▼
   DES
    │
    ▼
Ciphertext
```

### Main characteristics

| Feature            | DES                      |
| ------------------ | ------------------------ |
| Type               | Symmetric encryption     |
| Cipher type        | Block cipher             |
| Block size         | 64 bits                  |
| Key size           | 64 bits including parity |
| Effective key size | 56 bits                  |
| Number of rounds   | 16                       |
| Round key size     | 48 bits                  |
| Structure          | Feistel network          |

---

# 2. History of DES

DES has its origins in **IBM's Lucifer** cipher.

The original Lucifer design used a much larger key. During the standardization process, the key size was reduced.

The resulting algorithm was adopted by the U.S. government and standardized in **1977** as the **Data Encryption Standard**.

The important historical sequence is:

```text
IBM Lucifer
     │
     ▼
Modified design
     │
     ▼
DES / DEA
     │
     ▼
U.S. standard in 1977
```

DES became extremely popular and was used for many years in banking, financial systems, communications, and other applications.

However, its **56-bit effective key became too small** as computing power increased.

---

# 3. DES Block Size

DES processes data in blocks of exactly:

```text
64 bits
```

That means a long message is divided into 64-bit pieces.

For example:

```text
Large Message
      │
      ├── Block 1 → 64 bits
      ├── Block 2 → 64 bits
      ├── Block 3 → 64 bits
      └── Block 4 → 64 bits
```

Each 64-bit block is processed by the DES algorithm.

---

# 4. DES Key Size and Parity Bits

One confusing aspect of DES is that its key is often described as a **64-bit key**, while the actual cryptographic key length is only **56 bits**.

Why?

The 64-bit key contains **8 parity bits**.

```text
64-bit DES key
│
├── 56 bits → Actual key material
│
└── 8 bits  → Parity bits
```

There is one parity bit for every 8-bit group.

```text
8 bits   → 1 parity bit
8 bits   → 1 parity bit
8 bits   → 1 parity bit
...
```

Therefore:

```text
64 bits - 8 parity bits
        = 56 effective key bits
```

The parity bits are not used as part of the effective secret key.

---

# 5. Why the 56-Bit Key Became a Problem

A 56-bit key provides:

```text
2⁵⁶
```

possible key combinations.

That was extremely difficult to search using the computing technology available when DES was standardized.

However, computers became dramatically faster.

Eventually, specialized hardware could perform a **brute-force search** of the DES key space in a practical amount of time.

This was one of the main reasons DES was eventually replaced by stronger algorithms such as **AES**.

---

# 6. DES Encryption Overview

The DES encryption process can be divided into four major stages:

```text
64-bit Plaintext
       │
       ▼
┌─────────────────────┐
│ 1. Initial          │
│    Permutation (IP) │
└─────────┬───────────┘
          │
          ▼
    L₀        R₀
      \        /
       \      /
        ▼    ▼
     16 DES Rounds
          │
          ▼
       L₁₆ R₁₆
          │
          ▼
┌─────────────────────┐
│ 4. Final Permutation│
│       IP⁻¹          │
└─────────┬───────────┘
          │
          ▼
   64-bit Ciphertext
```

At the same time, the secret key goes through a **key schedule** to generate 16 round keys.

---

# 7. Step 1 — Initial Permutation (IP)

The first operation performed on the 64-bit plaintext is the **Initial Permutation (IP)**.

The input is:

```text
64-bit plaintext
```

DES rearranges these bits according to a predefined permutation table.

Importantly, the permutation does **not** change the actual bits.

It only changes their positions.

```text
Before:

10110010...

      │
      ▼
Initial Permutation

      │
      ▼

01001101...
```

The bits are rearranged according to the fixed IP table.

---

# 8. Splitting the Block

After the Initial Permutation, the 64-bit result is divided into two 32-bit halves:

```text
64-bit block
     │
     ├──────────────┐
     ▼              ▼
   32 bits        32 bits
     │              │
     ▼              ▼
    L₀             R₀
```

Where:

* `L₀` = Left half
* `R₀` = Right half

These two halves are then processed through **16 rounds**.

---

# 9. Step 2 — DES Key Schedule

DES needs a different **48-bit sub-key** for each of its 16 rounds.

Therefore:

```text
Original key
    │
    ▼
Key Schedule
    │
    ├── K₁
    ├── K₂
    ├── K₃
    ├── ...
    └── K₁₆
```

Each:

```text
Kₙ = 48 bits
```

So DES generates:

```text
16 × 48-bit round keys
```

from the effective 56-bit key.

---

# 10. Permuted Choice 1 — PC-1

The original DES key contains:

```text
64 bits
```

The first stage of the key schedule removes the 8 parity bits and rearranges the remaining bits using **Permuted Choice 1 (PC-1)**.

```text
64-bit key
    │
    ▼
   PC-1
    │
    ▼
56-bit key
```

The 56-bit result is divided into two halves:

```text
56 bits
   │
   ├─────────────┐
   ▼             ▼
28 bits         28 bits
   │             │
   ▼             ▼
  C₀             D₀
```

> **Note:** These halves are often called `C` and `D` in DES specifications to distinguish them from the plaintext halves `L` and `R`.

---

# 11. Left Circular Rotation

For every DES round, the `C` and `D` halves are shifted to the left.

This is a **circular left shift**.

For example:

```text
Before:

101101

Left rotation by 1:

011011
```

The bit that leaves the left side comes back at the right.

```text
C₀ ──► C₁
D₀ ──► D₁
```

The number of shifts depends on the particular DES round.

Some rounds use a one-bit rotation, while others use a two-bit rotation.

---

# 12. Permuted Choice 2 — PC-2

After rotation, the two 28-bit halves are combined:

```text
28 + 28
   │
   ▼
56 bits
```

The result is then processed using **Permuted Choice 2 (PC-2)**.

PC-2 selects and rearranges 48 bits.

```text
56-bit rotated key
       │
       ▼
      PC-2
       │
       ▼
48-bit round key
```

This produces:

```text
K₁
K₂
...
K₁₆
```

---

# 13. DES Key Schedule Summary

The entire key-generation process can be simplified as:

```text
64-bit Key
     │
     ▼
    PC-1
     │
     ▼
56-bit Key
     │
     ▼
Split into C₀ and D₀
     │
     ▼
Left Rotations
     │
     ▼
    PC-2
     │
     ▼
48-bit K₁
     │
     ▼
Next rotation
     │
     ▼
    PC-2
     │
     ▼
48-bit K₂
     │
     .
     .
     .
     ▼
48-bit K₁₆
```

---

# 14. Step 3 — The 16 DES Rounds

The main encryption process consists of **16 rounds**.

Each round receives:

```text
Lₙ₋₁
Rₙ₋₁
Kₙ
```

and produces:

```text
Lₙ
Rₙ
```

The basic Feistel equations are:

```text
Lₙ = Rₙ₋₁
```

and:

```text
Rₙ = Lₙ₋₁ ⊕ f(Rₙ₋₁, Kₙ)
```

where:

* `⊕` = XOR
* `f` = DES round/Mangler function
* `Kₙ` = round key

---

# 15. Understanding One DES Round

A single round can be visualized as:

```text
              Rₙ₋₁
                │
                ▼
           Expansion E
                │
                ▼
             48 bits
                │
                │ XOR Kₙ
                ▼
             48 bits
                │
                ▼
             S-Boxes
                │
                ▼
             32 bits
                │
                ▼
          Permutation P
                │
                ▼
                f
                │
                ▼
Lₙ₋₁ ───────── XOR ───────► Rₙ

Rₙ₋₁ ──────────────────────► Lₙ
```

This is the heart of DES.

---

# 16. The Mangler Function

The DES round function is commonly represented as:

```text
f(R, K)
```

It takes:

* a 32-bit right half
* a 48-bit round key

and produces:

```text
32-bit output
```

It consists of several important operations:

```text
32-bit R
   │
   ▼
Expansion
   │
   ▼
48 bits
   │
   ▼
XOR with K
   │
   ▼
48 bits
   │
   ▼
S-Boxes
   │
   ▼
32 bits
   │
   ▼
Permutation
   │
   ▼
32-bit output
```

---

# 17. Step 3.1 — Expansion (E)

The right half initially contains:

```text
32 bits
```

The expansion operation converts it into:

```text
48 bits
```

How?

Some bits are **duplicated** according to the DES expansion table.

```text
32-bit R
   │
   ▼
Expansion E
   │
   ▼
48-bit result
```

Why increase the size?

Because the result needs to be XORed with the:

```text
48-bit round key
```

---

# 18. Step 3.2 — XOR with the Round Key

The 48-bit expanded right half is XORed with the 48-bit round key.

```text
Expanded R
     │
     │ XOR
     ▼
   Kₙ
     │
     ▼
48-bit result
```

Mathematically:

```text
X = E(Rₙ₋₁) ⊕ Kₙ
```

The XOR operation combines the data with the secret key.

---

# 19. Step 3.3 — S-Boxes

This is one of the most important parts of DES.

The 48-bit result is divided into **eight 6-bit groups**.

```text
48 bits
   │
   ├── 6 bits → S-Box 1
   ├── 6 bits → S-Box 2
   ├── 6 bits → S-Box 3
   ├── 6 bits → S-Box 4
   ├── 6 bits → S-Box 5
   ├── 6 bits → S-Box 6
   ├── 6 bits → S-Box 7
   └── 6 bits → S-Box 8
```

Each S-Box converts:

```text
6 bits → 4 bits
```

Therefore:

```text
8 × 6 bits = 48 bits
```

becomes:

```text
8 × 4 bits = 32 bits
```

---

# 20. Why S-Boxes Are Important

S-Boxes provide an important source of **non-linearity** in DES.

Without strong non-linear transformations, attackers could potentially discover relationships between the plaintext, ciphertext, and key much more easily.

Conceptually:

```text
Input
  │
  ▼
S-Box
  │
  ▼
Unpredictable substitution
```

This contributes significantly to DES's **confusion** and resistance to cryptanalysis.

---

# 21. Step 3.4 — Permutation

After the S-Boxes produce 32 bits, DES performs another permutation.

```text
32-bit S-Box output
        │
        ▼
    P Permutation
        │
        ▼
32-bit round output
```

The permutation rearranges the bits.

This helps spread the influence of individual bits across subsequent rounds.

---

# 22. Complete DES Round Function

Putting the pieces together:

```text
             32-bit R
                 │
                 ▼
          Expansion E
                 │
                 ▼
             48 bits
                 │
                 │ XOR
                 ▼
             Round Key
                 │
                 ▼
             48 bits
                 │
                 ▼
           8 × S-Boxes
                 │
                 ▼
             32 bits
                 │
                 ▼
          P Permutation
                 │
                 ▼
              f(R,K)
```

This output is then XORed with the previous left half.

---

# 23. The Feistel Structure

DES uses a **Feistel network**.

A Feistel structure divides the block into two halves:

```text
        64-bit block
             │
       ┌─────┴─────┐
       ▼           ▼
      Left        Right
       │           │
       │           │
       │       Round Function
       │           │
       │           ▼
       └──── XOR ◄─┘
```

After each round, the halves effectively switch roles.

The key advantage of the Feistel structure is that the same general structure can be used for decryption.

---

# 24. One DES Round in Detail

Suppose we start with:

```text
L₀
R₀
```

Round 1 produces:

```text
L₁ = R₀

R₁ = L₀ ⊕ f(R₀, K₁)
```

Round 2:

```text
L₂ = R₁

R₂ = L₁ ⊕ f(R₁, K₂)
```

And so on:

```text
Round 1 → L₁, R₁
Round 2 → L₂, R₂
Round 3 → L₃, R₃
...
Round 16 → L₁₆, R₁₆
```

Each round increases the mixing of the plaintext and key.

---

# 25. All 16 DES Rounds

The complete structure looks like:

```text
L₀ R₀
 │  │
 ▼  ▼
Round 1 + K₁
 │
 ▼
L₁ R₁
 │
 ▼
Round 2 + K₂
 │
 ▼
L₂ R₂
 │
 ▼
Round 3 + K₃
 │
 ▼
   ...
 │
 ▼
Round 15 + K₁₅
 │
 ▼
L₁₅ R₁₅
 │
 ▼
Round 16 + K₁₆
 │
 ▼
L₁₆ R₁₆
```

After the sixteenth round, DES performs the final permutation.

---

# 26. Step 4 — Final Permutation

After the 16 rounds, DES has:

```text
L₁₆
R₁₆
```

These are combined to form a 64-bit block.

```text
L₁₆ || R₁₆
```

The result is then passed through the **Inverse Initial Permutation**:

```text
IP⁻¹
```

```text
L₁₆ || R₁₆
       │
       ▼
     IP⁻¹
       │
       ▼
64-bit Ciphertext
```

The Initial Permutation and Final Permutation are inverses of each other.

---

# 27. Complete DES Encryption

The complete process can now be visualized:

```text
                    64-bit Plaintext
                           │
                           ▼
                  Initial Permutation
                           │
                           ▼
                     L₀       R₀
                       \       /
                        \     /
                         ▼   ▼
                    ┌─────────────┐
                    │   Round 1   │ ← K₁
                    └──────┬──────┘
                           ▼
                    ┌─────────────┐
                    │   Round 2   │ ← K₂
                    └──────┬──────┘
                           ▼
                          ...
                           │
                    ┌─────────────┐
                    │  Round 16   │ ← K₁₆
                    └──────┬──────┘
                           ▼
                     L₁₆       R₁₆
                           │
                           ▼
                   Inverse Permutation
                           │
                           ▼
                   64-bit Ciphertext
```

At the same time:

```text
64-bit Key
    │
    ▼
   PC-1
    │
    ▼
56 bits
    │
    ▼
C₀ / D₀
    │
    ▼
Rotations
    │
    ▼
   PC-2
    │
    ▼
K₁ ... K₁₆
```

---

# 28. A Simple Mental Model

You do not need to memorize every DES permutation table to understand the algorithm.

Think of DES as:

```text
64-bit plaintext
       │
       ▼
   Rearrange bits
       │
       ▼
   Split in half
       │
       ▼
  ┌───────────────┐
  │ Repeat 16x    │
  │               │
  │ Expand        │
  │ XOR with key  │
  │ S-Boxes       │
  │ Permute       │
  │ XOR with half │
  └───────────────┘
       │
       ▼
 Recombine halves
       │
       ▼
 Final rearrangement
       │
       ▼
64-bit ciphertext
```

The key is simultaneously transformed into 16 different round keys.

---

# 29. Why DES Uses Multiple Rounds

One round is not enough to provide strong cryptographic mixing.

DES therefore repeats the transformation **16 times**.

Each round uses a different sub-key:

```text
K₁ → Round 1
K₂ → Round 2
K₃ → Round 3
...
K₁₆ → Round 16
```

Repeated rounds help create:

### Confusion

Makes the relationship between the key and ciphertext difficult to understand.

### Diffusion

Spreads the influence of individual plaintext bits throughout the ciphertext.

Together, these properties make statistical and structural attacks much more difficult.

---

# 30. DES Decryption

An important property of the Feistel structure is that decryption uses essentially the same structure as encryption.

The main difference is the order of the round keys.

Encryption:

```text
K₁ → K₂ → K₃ → ... → K₁₆
```

Decryption:

```text
K₁₆ → K₁₅ → K₁₄ → ... → K₁
```

This makes DES implementation relatively convenient.

```text
Encryption:
K₁ K₂ K₃ ... K₁₆

Decryption:
K₁₆ K₁₅ ... K₃ K₂ K₁
```

---

# 31. Why DES Is No Longer Secure

The main weakness of DES is its **56-bit effective key**.

There are:

```text
2⁵⁶
```

possible keys.

When DES was designed, searching this space was extremely expensive.

Modern computers and specialized hardware, however, can search much larger key spaces.

Therefore, DES is vulnerable to **brute-force attacks**.

Its historical importance is enormous, but:

> **DES should not be used for modern secure encryption.**

---

# 32. DES and Modern Cryptography

DES helped establish many important ideas that remain relevant today:

* Block cipher design
* Feistel networks
* S-Boxes
* Key schedules
* Multiple encryption rounds
* Confusion
* Diffusion
* Cryptanalysis
* Brute-force security analysis

DES was eventually replaced by stronger algorithms.

One transitional solution was **Triple DES (3DES)**, which applies DES multiple times.

Eventually, **AES** became the modern standard for symmetric encryption.

---

# 33. Important DES Terms

| Term            | Meaning                          |
| --------------- | -------------------------------- |
| DES             | Data Encryption Standard         |
| Block size      | 64 bits                          |
| Effective key   | 56 bits                          |
| Parity bits     | 8 bits in the nominal 64-bit key |
| Rounds          | 16                               |
| Round key       | 48 bits                          |
| IP              | Initial Permutation              |
| IP⁻¹            | Inverse Initial Permutation      |
| PC-1            | First key permutation            |
| PC-2            | Second key permutation           |
| S-Box           | 6-bit → 4-bit substitution       |
| E               | Expansion from 32 → 48 bits      |
| XOR             | Exclusive-OR operation           |
| Feistel network | Structure used by DES            |
| P               | Permutation after S-Boxes        |

---

# 34. DES at a Glance

```text
┌──────────────────────────────────────────┐
│                 DES                      │
├──────────────────────────────────────────┤
│ Symmetric Block Cipher                   │
│                                          │
│ Block size:       64 bits                │
│ Key size:         64 bits nominal        │
│ Effective key:   56 bits                │
│ Parity bits:       8 bits                │
│ Rounds:           16                     │
│ Round key:        48 bits                │
│ Structure:        Feistel                │
└──────────────────────────────────────────┘
```

### Encryption flow

```text
Plaintext
   │
   ▼
Initial Permutation
   │
   ▼
L₀ / R₀
   │
   ▼
16 Feistel Rounds
   │
   │ ← K₁ ... K₁₆
   ▼
L₁₆ / R₁₆
   │
   ▼
Inverse Initial Permutation
   │
   ▼
Ciphertext
```

### Key flow

```text
64-bit Key
   │
   ▼
Remove 8 parity bits
   │
   ▼
56 bits
   │
   ▼
Split into 28 + 28
   │
   ▼
Circular left shifts
   │
   ▼
PC-2
   │
   ▼
16 × 48-bit subkeys
```

---

# 35. Summary

If you need to remember DES for an exam, remember this sequence:

> **64 → IP → 32/32 → 16 rounds → IP⁻¹ → 64**

And for each round:

> **Expand → XOR Key → S-Boxes → Permutation → XOR → Swap**

The key schedule is:

> **64-bit key → PC-1 → 56 bits → 28/28 → Left rotations → PC-2 → 16 × 48-bit keys**

The most important numbers are:

```text
64-bit    → DES block size
64-bit    → nominal key size
56-bit    → effective key size
8 bits    → parity
16        → encryption rounds
48-bit    → round key
32-bit    → right half
8         → S-Boxes
6 → 4     → each S-Box
32 → 48   → expansion
```

**Most important conclusion:** DES was a landmark symmetric block cipher, but its **56-bit effective key is no longer sufficient against modern brute-force attacks**. Its design concepts—especially the Feistel structure, S-Boxes, key scheduling, confusion, and diffusion—remain fundamental to understanding modern block ciphers.
