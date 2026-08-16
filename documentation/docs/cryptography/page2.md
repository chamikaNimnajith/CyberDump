# Classical and Modern Symmetric Cryptography

This page continues the foundations of cryptography by examining **classical ciphers**, the information-theoretic principles behind modern cryptography, modern symmetric encryption, stream ciphers, randomness, and the cryptographic machines used during World War II.

Classical cryptography is important because many modern cryptographic concepts were developed as solutions to weaknesses found in older systems.

---

# 1. Classical Cryptography

**Classical cryptography** refers to cryptographic techniques developed before modern computer-based cryptography.

These ciphers are generally not secure today, but studying them helps us understand:

* How early encryption systems worked
* Why simple ciphers fail
* How cryptanalysis developed
* Why modern cryptographic algorithms need diffusion and confusion

The two major categories are:

```text
Classical Ciphers
       │
       ├── Substitution
       │
       └── Transposition
```

---

# 2. Substitution Ciphers

A **substitution cipher** replaces plaintext characters with other characters.

For example:

```text
Plaintext:  ABCDE
Ciphertext: BCDEF
```

The positions of the characters remain the same, but their values change.

---

## 2.1 Shift Cipher

A **shift cipher** moves each character by a fixed number of positions in the alphabet.

The Caesar Cipher is a famous example.

For a shift of 3:

```text
A → D
B → E
C → F
D → G
```

Therefore:

```text
HELLO
```

becomes:

```text
KHOOR
```

### Mathematical Representation

Characters can be represented using numbers:

```text
A = 0
B = 1
C = 2
...
Z = 25
```

Encryption can then be represented as:

```text
E(x) = (x + k) mod 26
```

where:

* `x` = plaintext character
* `k` = key/shift
* `26` = number of letters in the alphabet

Decryption is:

```text
D(x) = (x - k) mod 26
```

The use of modulo arithmetic makes the alphabet wrap around.

For example:

```text
Z + 1 = A
```

because:

```text
(25 + 1) mod 26 = 0
```

### Computer Variant

Computers work with bytes rather than alphabetic characters.

A similar concept can be applied using:

```text
E(x) = (x + k) mod 256
```

because a byte contains 256 possible values:

```text
0 – 255
```

However, simply shifting bytes does not provide secure modern encryption.

---

# 3. Affine Cipher

The **Affine Cipher** is a generalization of the shift cipher.

Instead of simply adding a key, it uses two parameters:

```text
(α, β)
```

The encryption formula is:

```text
E(x) = (αx + β) mod 26
```

where:

* `x` = plaintext character value
* `α` = multiplicative key
* `β` = additive key

For example, suppose:

```text
α = 5
β = 8
```

Then:

```text
E(x) = (5x + 8) mod 26
```

---

## 3.1 Why Must gcd(α, 26) = 1?

The value of `α` must have a **multiplicative inverse modulo 26**.

Therefore:

```text
gcd(α, 26) = 1
```

must be true.

For example:

```text
gcd(5, 26) = 1
```

so `5` can be used.

But:

```text
gcd(2, 26) = 2
```

so `2` cannot be used.

This condition is necessary because we need to reverse the multiplication during decryption.

---

## 3.2 Affine Decryption

The decryption formula is:

```text
D(y) = α⁻¹(y - β) mod 26
```

where:

```text
α⁻¹
```

is the **multiplicative inverse of α modulo 26**.

For example, if:

```text
α = 5
```

then:

```text
5 × 21 = 105
105 mod 26 = 1
```

Therefore:

```text
5⁻¹ mod 26 = 21
```

So the decryption process uses `21` to reverse the multiplication by `5`.

---

# 4. Monoalphabetic vs. Polyalphabetic Ciphers

Substitution ciphers can also be classified according to how many substitution alphabets they use.

## Monoalphabetic Cipher

A **monoalphabetic cipher** uses one substitution alphabet throughout the entire message.

The Caesar Cipher is an example.

```text
Plain A → Cipher D
Plain A → Cipher D
Plain A → Cipher D
```

The same plaintext character always produces the same ciphertext character.

### Weakness

This makes the cipher vulnerable to **frequency analysis**.

If a particular ciphertext character appears very frequently, an attacker may guess that it represents a common letter such as `E`.

---

## Polyalphabetic Cipher

A **polyalphabetic cipher** uses multiple substitution alphabets.

The Vigenère Cipher is a classic example.

The same plaintext letter can produce different ciphertext letters depending on its position and the key.

```text
Plaintext:
A → different ciphertext values
A → different ciphertext values
A → different ciphertext values
```

This makes simple frequency analysis more difficult.

---

# 5. Transposition Ciphers

A **transposition cipher** works differently from substitution.

Instead of replacing characters, it **rearranges their positions**.

For example:

```text
Plaintext:

HELLOWORLD
```

The letters themselves remain unchanged, but their positions may be rearranged.

```text
Ciphertext:

... rearranged letters ...
```

The key difference is:

| Cipher Type   | What Changes?       |
| ------------- | ------------------- |
| Substitution  | Character values    |
| Transposition | Character positions |

---

# 6. Columnar Transposition

In a **Columnar Transposition Cipher**, plaintext is written into a grid and then read according to the ordering of a key.

Suppose the key is:

```text
M A T H
```

The plaintext can be written across the columns:

```text
M   A   T   H
----------------
P   L   A   I
N   T   E   X
T   R   A   N
S   ...
```

The columns are then ordered according to the key.

The result is a ciphertext containing the same characters but in a different order.

The important concept is:

```text
No characters are replaced.
Only their positions change.
```

---

# 7. Double Transposition

**Double Transposition** applies transposition more than once.

A plaintext is first arranged in a two-dimensional structure and its rows or columns are rearranged.

The resulting text can then be transposed again using another ordering.

Conceptually:

```text
Plaintext
    ↓
First Transposition
    ↓
Intermediate Ciphertext
    ↓
Second Transposition
    ↓
Final Ciphertext
```

Double transposition is considerably stronger than a single simple transposition and was historically used for practical communication.

However, it is still not considered secure modern encryption.

---

# 8. Information Theory and Shannon

Modern cryptography was heavily influenced by the work of **Claude Shannon**.

Shannon introduced important concepts for understanding how a strong cipher should transform information.

Two particularly important principles are:

```text
Diffusion
Confusion
```

These help prevent attackers from discovering patterns in the plaintext or key.

---

# 9. Diffusion

**Diffusion** means spreading the statistical structure of the plaintext throughout the ciphertext.

The goal is that changing a small part of the plaintext should cause significant changes in the ciphertext.

For example:

```text
Plaintext 1:
HELLO WORLD

Plaintext 2:
HELLO WORLE
             ↑
          one change
```

A strong modern cipher should produce ciphertexts that are dramatically different.

This makes it difficult for attackers to use plaintext patterns or frequency analysis.

In modern block ciphers, this idea is related to the **avalanche effect**:

> A small change in the input should produce a large change in the output.

---

# 10. Confusion

**Confusion** makes the relationship between the key and ciphertext as complicated as possible.

The attacker should not be able to easily determine how individual key bits affect individual ciphertext bits.

Operations such as:

* Substitution
* XOR
* Bit shifting
* Permutations

can be combined to create complex relationships.

A simplified comparison is:

```text
Diffusion
    ↓
Hide plaintext patterns

Confusion
    ↓
Hide key-to-ciphertext relationships
```

Modern cryptographic algorithms use both principles extensively.

---

# 11. Modern Symmetric Cryptography

Modern symmetric cryptography follows an important principle:

> **The algorithm can be public; the key must remain secret.**

Modern algorithms are designed, standardized, publicly analyzed, and tested by cryptographers.

Examples include:

* AES
* ChaCha20

Modern symmetric encryption is generally divided into:

```text
Symmetric Cryptography
       │
       ├── Block Ciphers
       │
       └── Stream Ciphers
```

---

# 12. Block Ciphers

A **block cipher** processes data in fixed-size blocks.

For example:

```text
Plaintext
    ↓
┌──────────────┐
│ 128-bit block│
└──────────────┘
    ↓
Encryption
    ↓
┌──────────────┐
│ 128-bit block│
└──────────────┘
```

Historically, many block ciphers used 64-bit blocks.

Modern standards such as AES use:

```text
128-bit blocks
```

However, a block cipher by itself only explains how to encrypt one fixed-size block. Real messages can be much larger.

This is where **modes of operation** are required.

---

# 13. Modes of Operation

A mode of operation defines how a block cipher is used to encrypt messages longer than one block.

Common modes include:

* ECB
* CBC
* CFB
* OFB
* CTR

---

## 13.1 Electronic Codebook — ECB

In **ECB mode**, every plaintext block is encrypted independently.

```text
P1 → Encrypt → C1
P2 → Encrypt → C2
P3 → Encrypt → C3
```

The same plaintext block always produces the same ciphertext block when the same key is used.

Therefore:

```text
P1 = P3
```

results in:

```text
C1 = C3
```

### Why ECB Is Insecure

Repeated plaintext patterns remain visible in the ciphertext.

Conceptually:

```text
Plaintext:
AAAA BBBB AAAA

Ciphertext:
XXXX YYYY XXXX
```

The repeated pattern is preserved.

Therefore, **ECB should generally not be used for encrypting structured data**.

---

# 14. Cipher Block Chaining — CBC

CBC improves on ECB by chaining blocks together.

Before encryption, each plaintext block is XORed with the previous ciphertext block.

```text
P1 XOR IV → Encrypt → C1

P2 XOR C1 → Encrypt → C2

P3 XOR C2 → Encrypt → C3
```

The first block uses an **Initialization Vector (IV)** because there is no previous ciphertext block.

```text
IV → P1 → C1 → P2 → C2 → P3 → C3
```

---

## 14.1 Initialization Vector

An **Initialization Vector**, or IV, provides the initial value used by CBC.

The IV should be generated appropriately and must not be reused incorrectly with the same key.

Its purpose is to ensure that encrypting the same plaintext under the same key does not automatically produce the same ciphertext.

---

## 14.2 Padding

Block ciphers require complete blocks.

Suppose the block size is 128 bits but the message does not contain an exact multiple of 128 bits.

Padding can be added to fill the final block.

Examples include:

* Null-byte padding in certain contexts
* PKCS#7 padding

For example:

```text
Original:

HELLO
```

may be padded to fill the required block size.

Modern protocols need to handle padding carefully because incorrect padding handling can introduce vulnerabilities.

---

# 15. Other Block Cipher Modes

### Cipher Feedback — CFB

CFB converts a block cipher into a mode that can process smaller units of data.

### Output Feedback — OFB

OFB generates a keystream that can be XORed with plaintext.

### Counter Mode — CTR

CTR encrypts a sequence of counter values to generate a keystream.

Conceptually:

```text
Counter 1 → Encrypt → Keystream 1
Counter 2 → Encrypt → Keystream 2
Counter 3 → Encrypt → Keystream 3
```

The keystream is then XORed with the plaintext.

CTR allows encryption and decryption to be performed efficiently and can support parallel processing.

---

# 16. Data Encryption Standard — DES

**DES (Data Encryption Standard)** was one of the most historically important symmetric block ciphers.

It was based on IBM's **Lucifer** design.

DES operates on:

```text
Block size: 64 bits
Effective key size: 56 bits
```

Although the DES key is often described as 64 bits, 8 bits are used for parity, leaving **56 effective key bits**.

---

# 17. Breaking DES

As computing power increased, DES's 56-bit key became too small to provide adequate protection.

In 1975, **Whitfield Diffie and Martin Hellman** discussed the possibility of using large amounts of parallel computing power to perform exhaustive key searches.

Later, the **Electronic Frontier Foundation (EFF)** built a specialized machine known as **Deep Crack**.

In 1998, Deep Crack demonstrated that a DES key could be recovered in roughly three days.

This demonstrated an important principle:

> An encryption algorithm that was secure against available computing power at one point in history may become insecure as computing technology improves.

DES is now considered obsolete and should not be used for modern secure systems.

---

# 18. Triple DES — 3DES

Triple DES, or **3DES**, was introduced to extend the useful lifetime of DES.

Instead of applying DES once, it applies DES multiple times.

Conceptually:

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

Several keying arrangements were defined, including:

* DES-EEE3
* DES-EDE3
* DES-EEE2
* DES-EDE2

`E` represents encryption and `D` represents decryption.

Although 3DES was significantly stronger than single DES, it is now considered legacy technology and has been replaced by newer algorithms such as AES.

---

# 19. Advanced Encryption Standard — AES

**AES (Advanced Encryption Standard)** is one of the most important modern symmetric encryption standards.

It was selected by **NIST** based on the **Rijndael** algorithm.

AES supports:

```text
AES-128 → 128-bit key
AES-192 → 192-bit key
AES-256 → 256-bit key
```

AES uses:

```text
Block size: 128 bits
```

AES is widely used in:

* HTTPS/TLS
* VPNs
* Disk encryption
* Wireless security
* Secure applications
* Cloud systems

AES is designed to provide strong security while being efficient on modern computers.

---

# 20. International Data Encryption Algorithm — IDEA

**IDEA** is another symmetric block cipher.

Its characteristics include:

```text
Key size:   128 bits
Block size: 64 bits
Rounds:     8
```

IDEA combines several mathematical operations to produce encryption.

It was historically considered a strong alternative to DES and was used in systems such as older versions of PGP.

However, AES is much more commonly used in modern systems.

---

# 21. Stream Ciphers

A **stream cipher** encrypts data as a continuous stream rather than processing fixed-size blocks in the same way as a block cipher.

The basic idea is:

```text
Plaintext XOR Keystream = Ciphertext
```

For decryption:

```text
Ciphertext XOR Keystream = Plaintext
```

The keystream should appear random and should be unpredictable to an attacker.

Conceptually:

```text
Plaintext
    XOR
Keystream
    ↓
Ciphertext
```

Stream ciphers are useful when data needs to be processed continuously.

---

# 22. RC4

**RC4 (Rivest Cipher 4)** was a historically important stream cipher.

It supported variable-length keys and was widely used in technologies such as:

* SSL/TLS
* WEP
* Other older wireless and network protocols

However, serious weaknesses were discovered in RC4.

As a result, RC4 is now considered **insecure and obsolete**.

Modern systems should use secure alternatives.

---

# 23. One-Time Pad

The **One-Time Pad (OTP)** is a special type of stream cipher with a remarkable property:

> When used correctly, it provides information-theoretically perfect secrecy.

For this to be true, the key must satisfy all of the following:

1. It must be truly random.
2. It must be at least as long as the message.
3. It must never be reused.
4. It must remain secret.

The basic operation is:

```text
Plaintext XOR Random Key
          ↓
      Ciphertext
```

Decryption uses the same key:

```text
Ciphertext XOR Random Key
          ↓
       Plaintext
```

---

# 24. The Danger of Reusing an OTP Key

The biggest practical problem with OTP is key management.

Suppose:

```text
C1 = P1 XOR K
C2 = P2 XOR K
```

If the same key `K` is reused, an attacker can calculate:

```text
C1 XOR C2
```

which gives:

```text
(P1 XOR K) XOR (P2 XOR K)
```

Because:

```text
K XOR K = 0
```

the keys cancel:

```text
P1 XOR P2
```

The attacker therefore gains information about the two plaintexts.

---

# 25. VENONA Project

A famous historical example of OTP key reuse occurred during the **VENONA project**.

Some Soviet communications reused one-time-pad material.

This violated one of the most important OTP rules:

```text
Never reuse the key.
```

The reuse provided cryptanalysts with information that helped them decrypt portions of the messages.

This demonstrates that even a mathematically secure cryptographic system can fail when it is implemented or operated incorrectly.

---

# 26. Randomness and Keystream Generation

Randomness is extremely important in cryptography.

Cryptographic systems may need random values for:

* Keys
* Initialization vectors
* Nonces
* Salts
* Keystreams
* Cryptographic protocols

A weak random number generator can make an otherwise strong cryptographic algorithm vulnerable.

---

# 27. True Randomness vs. Pseudorandomness

Random values can be generated using physical processes.

Examples include:

* Electronic noise
* Thermal noise
* Other physical phenomena

These can provide sources of **true randomness**.

Computers also use algorithms called **pseudorandom number generators (PRGs)**.

A PRG starts with a seed:

```text
Seed
 ↓
PRG
 ↓
Pseudo-random sequence
```

The output appears random, but it is generated deterministically.

For cryptographic purposes, we need **cryptographically secure pseudorandom number generators (CSPRNGs)** rather than ordinary random generators.

---

# 28. Cryptographic Randomness

A cryptographic random generator should produce output that is:

* Difficult to predict
* Statistically suitable for its intended purpose
* Resistant to attackers who know some previous output
* Properly seeded

Randomness testing can examine whether generated data exhibits suspicious patterns.

However, passing statistical tests alone does not prove that a generator is cryptographically secure.

---

# 29. Linear Feedback Shift Registers — LFSR

A **Linear Feedback Shift Register (LFSR)** is a traditional method for generating pseudorandom-looking sequences.

It consists of:

* A sequence of bits
* Shift operations
* Feedback calculations

Conceptually:

```text
┌───┬───┬───┬───┬───┐
│ 1 │ 0 │ 1 │ 1 │ 0 │
└───┴───┴───┴───┴───┘
  ↑               │
  └── Feedback ───┘
```

LFSRs were historically used to generate hardware keystreams.

However, simple LFSR-based systems can be vulnerable to cryptanalysis because their mathematical structure can be exploited.

---

# 30. World War II Cryptographic Machines

World War II produced several famous cryptographic machines.

Important examples include:

* Enigma
* Purple
* SIGABA

These machines used mechanical or electromechanical components to implement complex substitution systems.

They can be thought of as early hardware-based cryptographic systems.

---

# 31. Enigma

The **Enigma machine** used rotating components called **rotors**.

A simplified structure looks like:

```text
Keyboard
   ↓
Plugboard
   ↓
Rotor 1
   ↓
Rotor 2
   ↓
Rotor 3
   ↓
Reflector
   ↓
Rotors
   ↓
Plugboard
   ↓
Output
```

Each key press caused the rotor positions to change.

This meant that the substitution changed continuously.

Therefore, Enigma was effectively a **polyalphabetic substitution system**.

---

# 32. Enigma Daily Settings

Enigma operators used daily settings distributed through codebooks.

These settings determined things such as:

* Rotor selection
* Rotor order
* Rotor positions
* Plugboard configuration

Therefore, the machine's behavior depended heavily on the daily configuration.

---

# 33. Enigma and Message Keys

To avoid simple statistical attacks, operators used procedures involving message-specific rotor settings.

The idea was to avoid encrypting every message using exactly the same starting state.

However, operational mistakes created weaknesses.

For example, repeated transmission of message-key information due to communication errors provided cryptanalysts with useful relationships between ciphertexts.

These weaknesses were exploited by Polish cryptanalysts and later by codebreakers at **Bletchley Park**.

---

# 34. Lessons from WWII Cryptography

The history of Enigma and other machines demonstrates an important security principle:

> **A complicated machine does not automatically produce secure cryptography.**

Security can fail because of:

* Poor algorithms
* Weak key management
* Predictable procedures
* Repeated keys
* Operator mistakes
* Statistical weaknesses
* Information leakage

Another important lesson is that protecting the physical design of a cryptographic system is not enough.

A system should remain secure even if attackers understand how the mechanism works.

---

# 35. Classical vs. Modern Cryptography

The evolution from classical ciphers to modern cryptography can be summarized as follows:

| Feature      | Classical Cryptography           | Modern Cryptography                     |
| ------------ | -------------------------------- | --------------------------------------- |
| Data         | Letters/symbols                  | Bits and bytes                          |
| Algorithms   | Often secret                     | Usually public                          |
| Keys         | Usually small                    | Very large                              |
| Security     | Often based on secrecy of method | Based on mathematical hardness          |
| Main attacks | Frequency analysis, brute force  | Cryptanalytic and computational attacks |
| Processing   | Manual/mechanical                | Computer-based                          |
| Examples     | Caesar, Vigenère, Enigma         | AES, modern stream ciphers              |

---

# 36. Key Takeaways

The most important concepts from this chapter are:

### Classical Cryptography

* **Shift ciphers** move characters using modular arithmetic.
* **Affine ciphers** combine multiplication and addition modulo 26.
* **Monoalphabetic ciphers** use one substitution alphabet.
* **Polyalphabetic ciphers** use multiple substitution alphabets.
* **Transposition ciphers** rearrange character positions.
* **Double transposition** applies transposition multiple times.

### Information Theory

* **Diffusion** spreads plaintext patterns throughout ciphertext.
* **Confusion** makes the relationship between the key and ciphertext complicated.

### Modern Symmetric Cryptography

* **Block ciphers** process fixed-size blocks.
* **Modes of operation** allow block ciphers to encrypt longer messages.
* **ECB** is insecure because repeated plaintext blocks produce repeated ciphertext blocks.
* **CBC** chains blocks using XOR and an IV.
* **DES** is obsolete because its 56-bit key is too small.
* **3DES** extended DES but is now legacy technology.
* **AES** is the major modern symmetric block cipher standard.
* **Stream ciphers** generate a keystream and combine it with plaintext.
* **RC4** is obsolete because of serious weaknesses.
* **One-Time Pads** can provide perfect secrecy when used correctly.
* **Randomness** is essential for secure cryptographic systems.

### WWII Cryptography

* Enigma and similar machines demonstrated the power and limitations of mechanical cryptography.
* Operational mistakes can defeat even complicated cryptographic systems.
* Security should depend on strong cryptographic principles rather than keeping the design secret.

The overall evolution can be viewed as:

```text
Classical Ciphers
       ↓
Cryptanalysis
       ↓
Shannon's Principles
       ↓
Modern Symmetric Cryptography
       ↓
Strong Algorithms + Large Keys
       ↓
Computer-Based Cryptographic Security
```
