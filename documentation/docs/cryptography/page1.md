# Foundations of Cryptography

Cryptography is the study of techniques used to protect information from unauthorized access, modification, or misuse. It is a fundamental part of cybersecurity and is used in technologies such as HTTPS, VPNs, digital signatures, password protection, secure messaging, and online banking.

This page introduces the fundamental concepts of cryptography, its history, common cipher types, key management, cryptographic attacks, and the impact of modern computing.

---

## 1. Foundational Concepts of Cryptography

Before studying advanced cryptography, it is important to understand a few basic terms.

### Plaintext

**Plaintext** is the original, readable information that we want to protect.

For example:

```text
HELLO WORLD
```

### Ciphertext

**Ciphertext** is the scrambled or encrypted form of plaintext.

For example:

```text
KHOOR ZRUOG
```

A person who does not have the required key should not be able to understand the ciphertext.

### Encryption

**Encryption** is the process of converting plaintext into ciphertext using an encryption algorithm and a key.

```text
Plaintext + Key
       ↓
   Encryption
       ↓
Ciphertext
```

### Decryption

**Decryption** is the process of converting ciphertext back into its original plaintext.

```text
Ciphertext + Key
       ↓
   Decryption
       ↓
Plaintext
```

---

## 2. Security Goals of Cryptography

Cryptography is not only about hiding information. It provides several important security properties.

### Confidentiality

**Confidentiality** ensures that unauthorized people cannot read the information.

For example, when you send a password over HTTPS, encryption helps prevent an attacker monitoring the network from reading it.

```text
Sender
  │
  │ Encrypted data
  ▼
Network ──────────► Receiver
          ↑
       Attacker
       cannot easily
       understand it
```

### Integrity

**Integrity** ensures that information has not been modified without authorization.

For example:

```text
Original message:
Transfer $100

Modified message:
Transfer $1000
```

Cryptographic mechanisms can help the receiver detect that the message was changed.

### Authentication

**Authentication** verifies the identity or origin of a communication.

For example, when connecting to a secure website, authentication mechanisms help verify that the server is actually the intended server.

### Non-repudiation

**Non-repudiation** provides evidence that a particular party performed an action or created a message.

Digital signatures are an important technology used to provide non-repudiation.

---

## 3. Cryptology

**Cryptology** is the broader field that includes both cryptography and cryptanalysis.

```text
                 Cryptology
                    │
          ┌─────────┴─────────┐
          │                   │
    Cryptography         Cryptanalysis
          │                   │
 Protect information    Analyze/break
 using cryptographic    cryptographic
 techniques             systems
```

### Cryptography

**Cryptography** focuses on designing techniques and algorithms for protecting information.

### Cryptanalysis

**Cryptanalysis** focuses on analyzing cryptographic systems and finding ways to recover protected information or keys without authorization.

---

# 4. Core Security Principles

## Kerckhoffs' Principle

One of the most important principles in cryptography is **Kerckhoffs' principle**.

The basic idea is:

> A cryptosystem should remain secure even when everything about the system, except the key, is publicly known.

In other words:

```text
Algorithm → Public
Key       → Secret
```

The security of the system should depend on the **secret key**, not on hiding the algorithm.

For example, AES is publicly documented and extensively analyzed by security researchers. Its security does not depend on keeping the AES algorithm secret.

---

## Security Through Obscurity

**Security through obscurity** means attempting to protect a system primarily by hiding how it works.

For example:

```text
"We will not publish our encryption algorithm,
so attackers cannot break it."
```

This is a weak security strategy.

Attackers can often:

* Reverse-engineer software
* Analyze network traffic
* Examine binaries
* Study implementation behavior
* Discover vulnerabilities

Therefore, a well-designed cryptographic system should remain secure even when its algorithm is publicly known.

---

## The Lock and Key Analogy

Cryptography can be compared to a physical lock.

Imagine a manufacturer produces a strong, well-tested lock. Everyone can see how the lock works, but only the person with the correct key can open it.

The important question is therefore:

```text
How difficult is it to obtain or guess the key?
```

The same principle applies to cryptography.

A strong cryptographic system should rely on:

* A strong algorithm
* A sufficiently large key space
* Proper key management

rather than simply hiding the algorithm.

---

# 5. History and Evolution of Cryptography

Cryptography has existed for thousands of years.

## Early Cryptography

Ancient civilizations used different techniques to hide or protect information.

One technique was **steganography**, which hides the existence of the message itself.

For example:

```text
Normal-looking document
        │
        └── Hidden message
```

The goal is not necessarily to encrypt the message, but to make other people unaware that a secret message exists.

---

## Caesar Cipher

The **Caesar Cipher** is one of the earliest well-known substitution ciphers.

It is associated with Julius Caesar, who reportedly used a shift of three positions.

For example:

```text
A → D
B → E
C → F
```

Therefore:

```text
HELLO
```

becomes:

```text
KHOOR
```

The letters are shifted by a fixed number of positions.

---

## Vigenère Cipher

The **Vigenère Cipher** was developed much later and introduced the idea of using multiple substitution shifts based on a key.

For example, instead of applying the same shift to every letter, different shifts are used according to the letters of a keyword.

This makes the cipher considerably more difficult to attack using simple frequency analysis than a basic monoalphabetic substitution cipher.

---

## World War II and Enigma

During World War II, machines such as the **Enigma** were used to encrypt military communications.

The complexity of machine-based cryptography helped push cryptography toward mathematics, engineering, and computing.

The efforts to break Enigma also contributed significantly to the development of modern computing and cryptanalysis.

---

# 6. Cipher Types and Mechanics

A **cipher** is an algorithm used to transform plaintext into ciphertext.

Traditional ciphers can broadly be divided into techniques such as:

* Substitution
* Transposition
* Symmetric encryption

---

## Symmetric Encryption

In **symmetric encryption**, the same secret key is used for encryption and decryption.

```text
             Secret Key
                 │
                 ▼
Plaintext → Encryption → Ciphertext
                              │
                              │
                         Decryption
                              │
                              ▼
                           Plaintext
```

The major challenge is securely sharing the secret key between the communicating parties.

Modern examples include:

* AES
* ChaCha20

---

# 7. Substitution Ciphers

A **substitution cipher** replaces characters or blocks with other characters or blocks.

For example:

```text
A → D
B → E
C → F
```

The original characters are replaced rather than rearranged.

---

## Caesar Cipher

The Caesar Cipher is a **monoalphabetic substitution cipher**.

With a shift of 3:

```text
Plain:   ABCDEFGHIJKLMNOPQRSTUVWXYZ
Cipher:  DEFGHIJKLMNOPQRSTUVWXYZABC
```

Therefore:

```text
HELLO
```

becomes:

```text
KHOOR
```

### Weakness

The Caesar Cipher has a very small key space.

An attacker can simply try all possible shifts.

Therefore, it is useful for understanding cryptographic concepts but is **not secure for modern applications**.

---

## Vigenère Cipher

The Vigenère Cipher uses a keyword to determine different substitution shifts.

For example:

```text
Plaintext:  ATTACK
Key:        KEYKEY
```

Each key character determines the shift used for the corresponding plaintext character.

Unlike the Caesar Cipher, the same plaintext letter may be encrypted into different ciphertext letters depending on the key position.

This makes basic frequency analysis more difficult.

However, the Vigenère Cipher is still vulnerable to cryptanalysis and should not be used for modern secure communication.

---

# 8. XOR

**XOR**, or Exclusive OR, is a fundamental operation in computer security and cryptography.

The XOR truth table is:

| A | B | A XOR B |
| - | - | ------- |
| 0 | 0 | 0       |
| 0 | 1 | 1       |
| 1 | 0 | 1       |
| 1 | 1 | 0       |

One useful property is:

```text
A XOR B XOR B = A
```

This means XOR can be used to demonstrate a simple symmetric encryption mechanism.

```text
Plaintext XOR Key = Ciphertext

Ciphertext XOR Key = Plaintext
```

For example:

```text
Plaintext  XOR Key = Ciphertext
Ciphertext XOR Key = Plaintext
```

However, simply using XOR with a weak or reused key does **not** automatically create secure encryption.

---

# 9. Transposition Ciphers

Unlike substitution ciphers, **transposition ciphers do not replace the characters**.

Instead, they rearrange their positions.

For example:

```text
PLAINTEXT
```

might be rearranged into:

```text
TEXTP... 
```

The exact arrangement depends on the cipher.

---

## Rail Fence Cipher

The **Rail Fence Cipher** writes characters in a zigzag pattern across multiple rows, or rails.

Conceptually:

```text
P     T     X
 L   N E   T
  A I   X
```

The characters are then read row by row to produce the ciphertext.

---

## Columnar Transposition

In a **Columnar Transposition Cipher**, the plaintext is written into columns and the columns are then rearranged according to a key.

The important difference is:

```text
Substitution:
Characters change

Transposition:
Character positions change
```

---

# 10. Symmetric vs. Asymmetric Cryptography

Modern cryptography uses both symmetric and asymmetric techniques.

## Symmetric Cryptography

Symmetric cryptography uses the **same secret key** for encryption and decryption.

```text
          Same Secret Key
          ┌─────────────┐
          ▼             ▼
Plaintext → Encrypt → Ciphertext
                      │
                      ▼
                   Decrypt
                      │
                      ▼
                   Plaintext
```

Advantages:

* Fast
* Efficient
* Suitable for large amounts of data

Disadvantage:

* Secure key distribution can be difficult

---

## Asymmetric Cryptography

Asymmetric cryptography uses a **key pair**:

* Public key
* Private key

The public key can generally be shared, while the private key must be protected.

```text
        Key Pair
       /        \
Public Key    Private Key
```

The two keys are mathematically related, but properly designed systems make deriving the private key from the public key computationally infeasible.

Examples include cryptographic systems based on:

* RSA
* Elliptic-curve cryptography

Asymmetric cryptography is generally slower than symmetric cryptography, so modern protocols often use both types together.

---

# 11. Key Length

The **key length** represents the number of bits in a cryptographic key.

For example:

```text
128-bit key
256-bit key
2048-bit key
3072-bit key
```

A larger key space generally makes brute-force attacks more difficult, but **key length cannot be compared directly across different algorithms**.

For example, a 128-bit AES key and a 128-bit RSA key do not provide equivalent security.

> Note: RSA keys are normally much larger than AES keys. A 512-bit RSA key is considered insecure today and should not be used for modern security applications.

---

# 12. Attacks on Cryptographic Systems

Cryptographic attacks attempt to discover secret information, recover keys, recover plaintext, or exploit weaknesses in implementations.

They can be divided into passive and active attacks.

---

## Passive Attacks

In a **passive attack**, the attacker observes information without modifying the communication.

Examples:

* Eavesdropping
* Network sniffing
* Traffic monitoring

Conceptually:

```text
Sender ───────────────► Receiver
             ▲
             │
          Attacker
          observes
```

The attacker attempts to learn information without changing the data.

---

## Active Attacks

In an **active attack**, the attacker modifies, injects, deletes, or otherwise interferes with data.

Examples include:

* Masquerading
* Message modification
* Replay attacks
* Message injection

Conceptually:

```text
Sender ─────► Attacker ─────► Receiver
                 │
                 └── modifies data
```

---

# 13. Cryptanalytic Attacks

## Ciphertext-Only Attack

In a **ciphertext-only attack**, the attacker has access only to ciphertext.

```text
Attacker knows:

Ciphertext
   ↓
?????
   ↓
Plaintext / Key
```

The attacker attempts to determine the plaintext or key from the available ciphertext.

---

## Known-Plaintext Attack

In a **known-plaintext attack**, the attacker knows some plaintext and its corresponding ciphertext.

```text
Known:

Plaintext  → Ciphertext
```

The attacker uses these known pairs to learn information about the encryption system or key.

---

## Chosen-Plaintext Attack

In a **chosen-plaintext attack**, the attacker can choose plaintext values and obtain their corresponding ciphertext.

```text
Attacker chooses plaintext
          │
          ▼
     Encryption
          │
          ▼
      Ciphertext
```

The attacker then analyzes the plaintext/ciphertext pairs to identify weaknesses.

Modern cryptographic algorithms are designed to withstand strong attack models such as chosen-plaintext attacks.

---

## Brute-Force Attack

A **brute-force attack** attempts every possible key until the correct one is discovered.

For an `n`-bit key, there are:

```text
2ⁿ
```

possible keys.

For example:

```text
8-bit key   → 2⁸   = 256 possibilities
16-bit key  → 2¹⁶  = 65,536 possibilities
128-bit key → 2¹²⁸ possibilities
```

The number of possibilities grows exponentially as the key length increases.

This is why modern cryptographic systems use sufficiently large key spaces.

---

# 14. Statistical Attacks

Statistical attacks take advantage of patterns in language.

For example, in English text, some letters occur more frequently than others.

An attacker can analyze:

* Individual letter frequencies
* Common pairs of letters
* Common groups of three letters

For example:

```text
Single letters → E, T, A, O, ...
Pairs          → TH, HE, IN, ER, ...
Triplets       → THE, AND, ING, ...
```

This approach can be effective against simple monoalphabetic substitution ciphers.

Modern encryption algorithms are specifically designed to avoid these kinds of simple statistical patterns.

---

# 15. The Influence of Modern Computing

Traditional cryptography often operated on letters and symbols.

Modern cryptography operates primarily on **binary data**.

Computers represent information as:

```text
Bits → 0 or 1
Bytes → groups of 8 bits
```

Modern cryptographic algorithms can process enormous amounts of data very quickly.

For example:

```text
Plaintext
    ↓
Binary representation
    ↓
Cryptographic algorithm
    ↓
Ciphertext
```

Computers also made it practical to use extremely large key spaces.

For example:

```text
2⁸
2¹⁶
2⁶⁴
2¹²⁸
2²⁵⁶
```

The difference between these numbers is enormous.

A key space of `2²⁵⁶` contains an astronomically large number of possible keys, making straightforward brute-force attacks computationally infeasible with currently known practical technology.

However, security does not depend only on key length. The algorithm, implementation, randomness, key management, protocols, and protection of secret keys are all important.

---

# 16. Putting Everything Together

A simplified view of cryptography is:

```text
                    Cryptology
                       │
              ┌────────┴────────┐
              │                 │
        Cryptography       Cryptanalysis
              │
       Protect Information
              │
      ┌───────┴────────┐
      │                │
   Symmetric        Asymmetric
      │                │
   Same Key        Public/Private
      │                │
      └───────┬────────┘
              │
       Security Goals
              │
   ┌──────────┼───────────┐
   │          │           │
Confidentiality Integrity Authentication
                         │
                  Non-repudiation
```

The central idea is simple:

> **Cryptography uses mathematical techniques to protect information, while cryptanalysis studies how those protections can be analyzed or broken.**

Understanding plaintext, ciphertext, keys, encryption, decryption, security goals, cipher types, and cryptanalytic attacks provides the foundation needed to study more advanced topics such as AES, RSA, ECC, hashing, digital signatures, certificates, TLS, and modern cryptographic protocols.
