# Modern Symmetric Cryptography: Stream Ciphers, Block Ciphers, DES & AES

Modern symmetric cryptography is one of the most important foundations of cybersecurity. It is used to protect data in network communication, wireless networks, storage systems, applications, and many other technologies.

This page explains how modern symmetric encryption works, the difference between stream and block ciphers, block cipher modes, RC4 and WEP vulnerabilities, DES, AES, and the principles used to design secure block ciphers.

---

# 1. Modern Symmetric Cryptography

**Symmetric cryptography** is a type of encryption where the communicating parties use the **same secret key**.

The basic process is:

```text
         Secret Key
             |
             ▼
Plaintext → Encryption → Ciphertext
                           |
                           ▼
                      Decryption
                           |
                           ▼
                      Plaintext
                           ▲
                           |
                     Secret Key
```

The same key is required to encrypt and decrypt the data.

### Important Principle

Modern cryptography follows **Kerckhoffs' principle**:

> The security of the system should depend on the secrecy of the key, not on hiding the algorithm.

Therefore:

```text
Algorithm → Public
Key       → Secret
```

A modern cryptographic algorithm should be publicly available for researchers to analyze and test.

For example, AES is completely public. Its security does not depend on hiding how AES works.

---

# 2. Why Public Algorithms Are Important

Suppose a company creates a secret encryption algorithm:

```text
Company's secret algorithm
          ↓
      Encryption
```

The company might assume:

> "Attackers don't know our algorithm, so they cannot break it."

This is unreliable because attackers can:

* Reverse-engineer software
* Analyze binaries
* Observe input and output
* Study network traffic
* Search for implementation weaknesses

A stronger approach is:

```text
Public algorithm
       +
Strong secret key
       =
Secure cryptographic system
```

Public algorithms can be examined by security researchers around the world.

---

# 3. Two Major Types of Symmetric Ciphers

Symmetric encryption can broadly be divided into two categories:

```text
             Symmetric Cryptography
                     │
           ┌─────────┴─────────┐
           │                   │
     Stream Cipher        Block Cipher
           │                   │
   Bits / bytes at a     Fixed-size blocks
        time
```

---

# 4. Stream Ciphers

A **stream cipher** encrypts data as a continuous stream.

Instead of waiting for a complete block, it can encrypt data one bit or byte at a time.

The basic operation is:

```text
Plaintext XOR Keystream
          ↓
      Ciphertext
```

For decryption:

```text
Ciphertext XOR Keystream
          ↓
       Plaintext
```

This works because:

```text
A XOR B XOR B = A
```

The important component is the **keystream**.

A secure stream cipher must generate a keystream that is:

* Unpredictable
* Sufficiently random
* Never improperly reused

---

# 5. Block Ciphers

A **block cipher** processes data in fixed-size blocks.

For example:

```text
Plaintext
   │
   ├── Block 1
   ├── Block 2
   ├── Block 3
   └── Block 4
        │
        ▼
    Encryption
        │
        ▼
   Ciphertext
```

Historically, many block ciphers used **64-bit blocks**.

Modern algorithms such as AES use:

```text
128-bit blocks
```

The block cipher itself operates on one block, so additional mechanisms called **modes of operation** are required to encrypt messages of arbitrary length.

---

# 6. Stream Ciphers vs. Block Ciphers

| Feature                 | Stream Cipher            | Block Cipher                      |
| ----------------------- | ------------------------ | --------------------------------- |
| Processing              | Bit/byte stream          | Fixed-size blocks                 |
| Data handling           | Continuous               | Block-based                       |
| Typical operation       | XOR with keystream       | Block transformation              |
| Padding                 | Usually unnecessary      | May be required depending on mode |
| Example                 | RC4                      | AES, DES                          |
| Historical hardware use | Common                   | Also possible                     |
| Modern use              | Specialized applications | Very widespread                   |

There is also some overlap.

For example, certain block cipher modes can make a block cipher behave similarly to a stream cipher.

---

# 7. Why Block Ciphers Are Common Today

Modern processors can implement block ciphers such as AES extremely efficiently.

AES also has hardware acceleration on many modern CPUs.

Therefore, block ciphers are widely used in:

* HTTPS/TLS
* VPNs
* Disk encryption
* Cloud storage
* Applications
* Wireless security

However, stream-oriented designs can still be useful in environments where:

* Memory is limited
* Processing power is limited
* Data arrives continuously
* Low latency is important

Hardware implementations using structures such as **Linear Feedback Shift Registers (LFSRs)** have historically been useful in resource-constrained systems.

---

# 8. Block Cipher Modes of Operation

A block cipher normally accepts a fixed-size input.

For example:

```text
AES:

128-bit plaintext
       ↓
     AES
       ↓
128-bit ciphertext
```

But real messages can be:

```text
10 bytes
500 bytes
1 MB
10 GB
```

Therefore, we need a **mode of operation**.

A mode defines how multiple blocks are processed.

Important modes include:

* ECB
* CBC
* CFB
* OFB
* CTR

---

# 9. Electronic Codebook Mode — ECB

ECB encrypts each block independently.

```text
P1 → AES → C1
P2 → AES → C2
P3 → AES → C3
```

The major problem is:

```text
Same plaintext block
        ↓
Same key
        ↓
Same ciphertext block
```

Therefore:

```text
P1 = P3
```

results in:

```text
C1 = C3
```

This leaks patterns.

### Example

Imagine an image is encrypted using ECB.

Large areas containing identical pixel patterns can produce identical ciphertext blocks.

The encrypted image may therefore still reveal structural information about the original.

Therefore:

> **ECB should generally not be used for encrypting structured data.**

---

# 10. Cipher Block Chaining — CBC

CBC introduces a dependency between neighboring blocks.

The first plaintext block is XORed with an **Initialization Vector (IV)**.

```text
          IV
           │
           ▼
P1 ── XOR ──→ AES ──→ C1
                │
                │
P2 ── XOR ◄─────C1
                │
                ▼
               AES
                │
                ▼
               C2
```

The general formula is:

```text
C1 = E(P1 XOR IV)

C2 = E(P2 XOR C1)

C3 = E(P3 XOR C2)
```

This means each plaintext block depends on the previous ciphertext block.

---

# 11. Initialization Vector — IV

The first block has no previous ciphertext block.

Therefore CBC requires an IV.

```text
IV → P1 → C1 → P2 → C2 → P3 → C3
```

The IV should be generated appropriately and should not be improperly reused with the same key.

The IV does not need to be a secret value in the same way the encryption key is.

Its purpose is to provide a different starting state.

---

# 12. Padding in CBC

Block ciphers require complete blocks.

Suppose a cipher has a 128-bit block size, but the final message does not contain a complete 128-bit block.

Padding can be added.

For example:

```text
Message
   ↓
Add padding
   ↓
Complete block
   ↓
Encrypt
```

One common padding mechanism is **PKCS#7 padding**.

Incorrect handling of CBC padding can introduce serious vulnerabilities, so modern applications need to implement it carefully.

---

# 13. Cipher Feedback — CFB

**CFB** allows a block cipher to operate in a stream-like manner.

Instead of directly encrypting each plaintext block independently, the previous ciphertext information is used to generate the next encryption input.

Conceptually:

```text
Previous Ciphertext
        ↓
      Encrypt
        ↓
    Keystream
        ↓
   XOR with Plaintext
        ↓
     Ciphertext
```

CFB can process smaller units of data and does not require conventional padding in the same way as basic CBC.

---

# 14. Output Feedback — OFB

OFB also converts a block cipher into a stream-like construction.

The output of the encryption process is repeatedly fed back into the encryption process.

```text
Initial Value
     ↓
   Encrypt
     ↓
Keystream 1
     ↓
   Encrypt
     ↓
Keystream 2
     ↓
   Encrypt
     ↓
Keystream 3
```

The generated keystream is XORed with plaintext.

A major property of OFB is that transmission errors do not propagate through the entire remaining message in the same way they can in some chaining modes.

---

# 15. Counter Mode — CTR

CTR mode generates a keystream by encrypting counter values.

```text
Counter 1 → Encrypt → Keystream 1
Counter 2 → Encrypt → Keystream 2
Counter 3 → Encrypt → Keystream 3
```

Then:

```text
Plaintext 1 XOR Keystream 1 → Ciphertext 1
Plaintext 2 XOR Keystream 2 → Ciphertext 2
Plaintext 3 XOR Keystream 3 → Ciphertext 3
```

One major advantage of CTR is **parallelization**.

The blocks can be processed independently:

```text
Counter 1 ──→ Encryption ──→ Keystream 1
Counter 2 ──→ Encryption ──→ Keystream 2
Counter 3 ──→ Encryption ──→ Keystream 3
```

This can improve performance on modern processors.

---

# 16. Parallelization

Parallelization is an important consideration when designing encryption systems.

Some modes require blocks to be processed sequentially.

For example, CBC encryption has a dependency:

```text
C1
 ↓
P2
 ↓
C2
 ↓
P3
 ↓
C3
```

Therefore, the encryption of `P2` depends on `C1`.

CTR is different:

```text
Counter 1 → Encrypt
Counter 2 → Encrypt
Counter 3 → Encrypt
Counter 4 → Encrypt
```

These operations can be performed independently.

This makes CTR suitable for high-performance environments.

Parallelizable modes can also be useful when data arrives in pieces or packets that need to be processed independently.

---

# 17. Error Propagation

Different modes behave differently when ciphertext is corrupted.

For example, changing a bit in a ciphertext block can affect:

* Only the corresponding plaintext block
* Part of the next block
* Multiple blocks

depending on the mode.

This is known as **error propagation**.

Understanding error propagation is important for network communication because transmitted data can occasionally become corrupted.

---

# 18. RC4 Stream Cipher

**RC4 (Rivest Cipher 4)** was designed by **Ron Rivest**.

It became one of the most widely deployed stream ciphers in history.

It was used in technologies such as:

* SSL/TLS
* WEP
* Other network protocols

However, serious weaknesses were eventually discovered, and RC4 is now considered **obsolete and insecure**.

---

# 19. RC4 Internal State

RC4 maintains an internal state represented by an array containing **256 bytes**.

Conceptually:

```text
S = [
  S[0],
  S[1],
  S[2],
  ...
  S[255]
]
```

The array is initialized and rearranged using the secret key.

The algorithm then continually modifies the internal state to produce a keystream.

Simplified:

```text
Secret Key
    ↓
Key Scheduling
    ↓
256-byte State
    ↓
Pseudo-Random Generation
    ↓
Keystream
```

The keystream is then XORed with plaintext.

---

# 20. RC4 Key Scheduling

RC4 has a key scheduling algorithm commonly called **KSA**.

The key is used to initialize and permute the 256-byte state.

Then the **Pseudo-Random Generation Algorithm (PRGA)** produces the keystream.

```text
Key
 ↓
KSA
 ↓
Initial State
 ↓
PRGA
 ↓
Keystream
```

The security of RC4 depends heavily on the statistical properties of this generated keystream.

Unfortunately, RC4 has significant biases.

---

# 21. FMS Attack

One important attack against RC4 is the **Fluhrer, Mantin, and Shamir (FMS) attack**.

The attack became particularly important against **WEP**.

The problem was not simply that RC4 existed.

Instead, weaknesses resulted from the combination of:

* RC4's key scheduling weaknesses
* WEP's small 24-bit IV
* Predictable packet structures
* Reuse of RC4 key material

By collecting enough packets with appropriate IVs, an attacker could statistically recover information about the secret key.

---

# 22. Wired Equivalent Privacy — WEP

**WEP** was an early security protocol designed for wireless networks.

It attempted to provide confidentiality and integrity using:

```text
RC4
+
24-bit IV
+
CRC checksum
```

The basic idea was:

```text
Secret Key + IV
       ↓
      RC4
       ↓
   Keystream
       ↓
Plaintext XOR Keystream
       ↓
   Ciphertext
```

The IV was transmitted along with the encrypted packet.

---

# 23. WEP's 24-bit IV Problem

WEP used a **24-bit initialization vector**.

That means there are:

```text
2²⁴
```

possible IV values.

This is a relatively small space for a busy wireless network.

Eventually, IVs are reused.

If the same key and IV combination are reused, the same keystream can be generated.

This is extremely dangerous for a stream cipher.

---

# 24. Why WEP Packets Leak Information

Wireless protocols contain predictable information.

For example, an attacker can capture packets containing predictable structures such as ARP traffic.

The attacker may know part of the plaintext.

If:

```text
Ciphertext = Plaintext XOR Keystream
```

then:

```text
Keystream = Ciphertext XOR Plaintext
```

Therefore, if an attacker knows both:

```text
Ciphertext
Plaintext
```

they can recover the corresponding keystream bytes.

This information can then contribute to statistical attacks against WEP.

---

# 25. WEP and ARP Traffic

ARP requests and responses are particularly useful because their structure is predictable.

An attacker can capture many packets and identify portions of their plaintext structure.

Conceptually:

```text
Known plaintext
       +
Captured ciphertext
       ↓
Recover keystream information
       ↓
Collect many samples
       ↓
Statistical analysis
       ↓
Recover WEP key
```

This is one of the reasons WEP was fundamentally unsuitable for modern wireless security.

---

# 26. Active Packet Injection

WEP attacks were not limited to passive observation.

Attackers could also perform **active packet injection**.

The general idea is:

```text
Attacker
   │
   │ Inject packet
   ▼
Wireless Network
   │
   └── Generates response traffic
             ↓
       More packets available
             ↓
       More cryptographic data
```

By causing additional traffic, attackers could accelerate the collection of useful packets.

This significantly reduced the practical time required to recover WEP keys.

---

# 27. WEP Lessons

WEP demonstrates several important cryptographic lessons:

* Do not use a small IV space.
* Do not reuse keystreams.
* Do not rely on weak integrity mechanisms.
* Do not use outdated cryptographic algorithms.
* Secure protocols must consider both passive and active attackers.

WEP has been replaced by stronger wireless security standards.

---

# 28. Data Encryption Standard — DES

**DES (Data Encryption Standard)** is a historically important block cipher.

Its main characteristics are:

```text
Block size: 64 bits
Effective key: 56 bits
Rounds: 16
```

DES processes a 64-bit plaintext block and produces a 64-bit ciphertext block.

```text
64-bit Plaintext
       ↓
      DES
       ↓
64-bit Ciphertext
```

Although the original key representation is 64 bits, 8 bits are used for parity, resulting in an effective key size of 56 bits.

---

# 29. DES Feistel Structure

DES uses a **Feistel structure**.

The plaintext block is divided into two halves:

```text
64-bit block
     │
 ┌───┴───┐
 │       │
Left    Right
32-bit  32-bit
```

During each round, a function is applied to one half and combined with the other half.

A simplified Feistel round looks like:

```text
Lᵢ₊₁ = Rᵢ

Rᵢ₊₁ = Lᵢ XOR F(Rᵢ, Kᵢ)
```

where:

* `L` = left half
* `R` = right half
* `F` = round function
* `Kᵢ` = round key

This structure allows encryption and decryption to use closely related procedures.

---

# 30. DES Round Function

The DES round function contains several transformations.

A simplified view is:

```text
Right Half
    ↓
Expansion
    ↓
XOR with Round Key
    ↓
S-Boxes
    ↓
Permutation
    ↓
Round Output
```

### S-Boxes

**Substitution boxes**, or S-boxes, introduce nonlinear transformations.

They are important for creating **confusion**.

### Permutations

The permutation operations rearrange bits and help create **diffusion**.

Together, these transformations make it difficult to identify relationships between plaintext, ciphertext, and the key.

---

# 31. DES and Cryptanalysis

DES was heavily studied by cryptographers.

Researchers investigated:

* Brute-force attacks
* Differential cryptanalysis
* Linear cryptanalysis
* Other analytical techniques

The major practical problem eventually became the 56-bit key.

As computing power increased, exhaustive key searches became increasingly practical.

---

# 32. Differential Cryptanalysis

**Differential cryptanalysis** studies how differences in plaintext inputs affect differences in ciphertext outputs.

Conceptually:

```text
Plaintext A ──→ DES ──→ Ciphertext A
       │
       │ Difference
       ▼
Plaintext B ──→ DES ──→ Ciphertext B
```

An attacker studies relationships between:

```text
Input differences
        ↓
Output differences
```

This can reveal information about the internal structure or key.

Modern block ciphers are designed specifically to resist differential cryptanalysis.

---

# 33. Triple DES

As DES became weaker because of its small key size, **Triple DES (3DES)** was introduced.

The basic idea is to apply DES multiple times:

```text
Plaintext
   ↓
DES
   ↓
DES
   ↓
DES
   ↓
Ciphertext
```

3DES provided much greater security than single DES.

However, it is now considered legacy technology and has largely been replaced by AES.

---

# 34. Advanced Encryption Standard — AES

**AES**, or Advanced Encryption Standard, is one of the most important modern symmetric block ciphers.

AES is based on the **Rijndael** algorithm.

AES uses:

```text
Block size: 128 bits
```

It supports three key sizes:

| Version | Key Size | Number of Rounds |
| ------- | -------: | ---------------: |
| AES-128 | 128 bits |               10 |
| AES-192 | 192 bits |               12 |
| AES-256 | 256 bits |               14 |

The block size remains **128 bits** for all three versions.

---

# 35. AES State Array

AES represents a 128-bit block as a **4 × 4 array of bytes**, called the state.

Conceptually:

```text
┌────┬────┬────┬────┐
│ B0 │ B1 │ B2 │ B3 │
├────┼────┼────┼────┤
│ B4 │ B5 │ B6 │ B7 │
├────┼────┼────┼────┤
│ B8 │ B9 │ B10│ B11│
├────┼────┼────┼────┤
│ B12│ B13│ B14│ B15│
└────┴────┴────┴────┘
```

Each cell contains one byte.

Therefore:

```text
16 bytes × 8 bits = 128 bits
```

---

# 36. AES Round Transformations

AES uses several major transformations:

```text
SubBytes
ShiftRows
MixColumns
AddRoundKey
```

These transformations provide confusion and diffusion.

---

# 37. SubBytes

**SubBytes** replaces each byte in the state using a predefined substitution table called the **S-box**.

Conceptually:

```text
Input Byte
    ↓
 AES S-Box
    ↓
Output Byte
```

For example:

```text
Byte A → S-Box → Byte B
```

The S-box introduces nonlinear behavior.

This helps resist cryptanalytic attacks.

---

# 38. ShiftRows

**ShiftRows** rotates the rows of the AES state.

Conceptually:

```text
Before:

A B C D
E F G H
I J K L
M N O P
```

After shifting:

```text
A B C D
F G H E
K L I J
P M N O
```

The exact transformation is defined by the AES specification.

The purpose is to spread bytes across different columns.

---

# 39. MixColumns

**MixColumns** transforms each column of the state using mathematical operations in a finite field.

Conceptually:

```text
Column
  │
  ▼
MixColumns
  │
  ▼
Different column
```

It causes bytes from the same column to influence one another.

Together with ShiftRows, this produces strong diffusion throughout the state.

---

# 40. AddRoundKey

In **AddRoundKey**, the current state is combined with a round key using XOR.

```text
State XOR Round Key
        ↓
   New State
```

This operation introduces the secret key into the encryption process.

The overall idea is:

```text
State
  ↓
SubBytes
  ↓
ShiftRows
  ↓
MixColumns
  ↓
AddRoundKey
  ↓
Next Round
```

The final AES round omits MixColumns.

---

# 41. AES Key Expansion

AES does not simply use the original key unchanged in every round.

Instead, the original key is expanded into a collection of **round keys**.

```text
Original Key
     ↓
Key Expansion
     ↓
Round Key 1
Round Key 2
Round Key 3
...
Round Key N
```

Each encryption round uses the appropriate round key.

This makes the relationship between the original secret key and the ciphertext more complex.

---

# 42. DES vs. AES

| Feature                  | DES        | AES                                                                 |
| ------------------------ | ---------- | ------------------------------------------------------------------- |
| Block size               | 64 bits    | 128 bits                                                            |
| Effective key            | 56 bits    | 128/192/256 bits                                                    |
| Structure                | Feistel    | Substitution-permutation                                            |
| Rounds                   | 16         | 10/12/14                                                            |
| Status                   | Obsolete   | Modern standard                                                     |
| Security                 | Inadequate | Strong when used correctly                                          |
| Main historical weakness | Small key  | No comparable practical brute-force weakness for standard key sizes |

AES is significantly more appropriate for modern systems.

---

# 43. Modern Block Cipher Design

Modern block ciphers must be designed to resist many types of attacks.

Two important categories are:

### Differential Cryptanalysis

Studies relationships between differences in plaintext and ciphertext.

### Linear Cryptanalysis

Attempts to approximate relationships between plaintext, ciphertext, and key bits using linear equations.

A strong block cipher should resist both.

---

# 44. Security vs. Performance

Cryptographic algorithms must balance:

```text
Security
   ↕
Performance
   ↕
Implementation Complexity
   ↕
Resource Requirements
```

A theoretically secure algorithm may be impractical if it is extremely slow or requires too much memory.

Modern algorithms therefore aim to provide:

* Strong cryptographic security
* Efficient software implementation
* Efficient hardware implementation
* Resistance to known cryptanalytic techniques
* Reasonable memory usage
* High throughput

---

# 45. Overall Picture

The evolution of symmetric cryptography can be summarized as:

```text
Classical Ciphers
       ↓
Simple Substitution
       ↓
Polyalphabetic Systems
       ↓
Mechanical Cryptographic Machines
       ↓
DES
       ↓
3DES
       ↓
AES
       ↓
Modern Cryptographic Systems
```

The development of modern cryptography was driven by the discovery of weaknesses in older systems.

The important progression is:

```text
Small Keys
   ↓
Large Key Spaces

Simple Patterns
   ↓
Diffusion + Confusion

Secret Algorithms
   ↓
Publicly Analyzed Algorithms

Independent Blocks
   ↓
Secure Modes of Operation

Weak Randomness
   ↓
Cryptographically Secure Randomness
```

---

# 46. Key Takeaways

### Modern Symmetric Cryptography

* Symmetric encryption uses the same secret key for encryption and decryption.
* The algorithm should be public; the key should remain secret.
* Symmetric ciphers are broadly divided into stream and block ciphers.

### Stream Ciphers

* Process data as a continuous stream.
* Usually combine plaintext with a keystream using XOR.
* RC4 was historically important but is now insecure.
* Keystream reuse can completely undermine security.

### WEP

* Used RC4 and a 24-bit IV.
* The small IV space caused reuse.
* Predictable traffic such as ARP made attacks easier.
* FMS and related attacks demonstrated serious weaknesses.
* Active packet injection could accelerate attacks.
* WEP is obsolete.

### Block Ciphers

* Process fixed-size blocks.
* Require appropriate modes for arbitrary-length messages.
* ECB leaks repeated patterns.
* CBC uses chaining and an IV.
* CTR supports efficient parallel processing.

### DES

* Uses 64-bit blocks and a 56-bit effective key.
* Uses a 16-round Feistel structure.
* Uses S-boxes and permutations to provide confusion and diffusion.
* Became vulnerable to brute-force attacks.
* 3DES extended DES but is now legacy technology.

### AES

* Uses 128-bit blocks.
* Supports 128-, 192-, and 256-bit keys.
* Uses 10, 12, or 14 rounds.
* Its major transformations are **SubBytes, ShiftRows, MixColumns, and AddRoundKey**.
* AES is a major modern standard for symmetric encryption.

### Modern Cipher Design

A strong modern block cipher should:

```text
Provide strong confusion
          +
Provide strong diffusion
          +
Resist differential cryptanalysis
          +
Resist linear cryptanalysis
          +
Use sufficiently large keys
          +
Be efficient to implement
```

The central lesson is that **modern cryptography is not simply about making data look random**. It requires carefully designed mathematical structures that remain secure even when attackers know the algorithm and can perform extensive analysis.
