# Modern Cryptography: Authentication, Integrity, Hashing & Non-Repudiation

Modern cryptography is not limited to encrypting information.

In real-world systems, we need to answer several important questions:

* **Can unauthorized people read this message?**
* **Did this message really come from the claimed sender?**
* **Was the message changed during transmission?**
* **Can the sender later deny sending the message?**

Modern cryptographic systems address these requirements through **confidentiality, authentication, integrity, and non-repudiation**.

This page explains these concepts, challenge-response authentication, cryptographic hashes, MACs, HMACs, digital signatures, collision attacks, and the history of major hash algorithms.

---

# 1. Uses of Modern Cryptography

Modern cryptography provides four major security properties:

```text
                Modern Cryptography
                       │
       ┌───────────────┼────────────────┐
       │               │                │
Confidentiality  Authentication     Integrity
                                        │
                                 Non-repudiation
```

---

## 1.1 Confidentiality

**Confidentiality** means preventing unauthorized people from reading information.

Encryption is the primary mechanism used to provide confidentiality.

```text
Plaintext
    ↓
Encryption + Key
    ↓
Ciphertext
    ↓
Decryption + Key
    ↓
Plaintext
```

Examples include:

* AES
* TLS encryption
* Disk encryption
* VPN encryption

For example:

```text
Alice → "Transfer $100"
             ↓
          Encrypt
             ↓
       Unreadable data
             ↓
           Bob
```

An attacker who intercepts the ciphertext should not be able to understand the original message without the required key.

---

# 2. Authentication

**Authentication** means verifying the identity or origin of something.

For example:

> "How do I know this message actually came from Alice?"

Authentication is different from confidentiality.

A message can be encrypted but still require authentication.

```text
Confidentiality:
"Can someone read it?"

Authentication:
"Who sent it?"
```

Common authentication mechanisms include:

* Passwords
* Shared secret keys
* Digital certificates
* Digital signatures
* Challenge-response protocols

---

# 3. Integrity

**Integrity** means ensuring that information has not been modified without authorization.

Suppose Alice sends:

```text
Transfer $100 to Bob
```

An attacker changes it to:

```text
Transfer $1000 to Trudi
```

The receiver needs a way to detect this modification.

```text
Original Message
      ↓
Integrity Protection
      ↓
Transmission
      ↓
Integrity Verification
      ↓
Modified?
  ┌───┴───┐
 No      Yes
 ↓        ↓
Accept   Reject
```

---

# 4. Non-Repudiation

**Non-repudiation** provides evidence that a particular party performed an action or sent a message.

For example:

> Alice signs a digital contract.

Later Alice says:

> "I never signed that contract."

A properly implemented digital-signature system can provide evidence that the signature was created using Alice's private key.

This property is particularly important for:

* Digital contracts
* Financial transactions
* Legal documents
* Software signing
* Electronic approvals

---

# 5. Authentication Using Shared Secrets

One simple approach to authentication is for two parties to share a secret key.

For example:

```text
Alice ─────── Shared Secret ─────── Bob
```

Alice can demonstrate knowledge of the secret without necessarily sending the secret itself.

A simplified approach might involve encrypting or authenticating a known value.

If Bob can verify the result using the shared secret, Bob gains evidence that Alice knows the secret.

However, simply sending a static encrypted value is not sufficient for strong authentication because an attacker could potentially capture and replay it.

This leads to **challenge-response authentication**.

---

# 6. Challenge-Response Authentication

A **challenge-response protocol** allows one party to prove knowledge of a secret without directly sending the secret.

Suppose Alice and Bob share a secret key.

```text
Alice                    Bob
  │                       │
  │       Challenge       │
  │ <──────────────────── │
  │                       │
  │       Response        │
  │ ────────────────────> │
  │                       │
  │     Authentication    │
  │        succeeds       │
```

The challenge is normally a random value called a **nonce**.

---

# 7. What Is a Nonce?

A **nonce** is a value intended to be used once.

For example:

```text
Nonce = 839271
```

Bob generates a random nonce and sends it to Alice.

Alice uses the shared secret to calculate an authentication response.

For example:

```text
Response = MAC(SecretKey, Nonce)
```

Alice sends the response to Bob.

Bob calculates the same value:

```text
MAC(SecretKey, Nonce)
```

If the values match, Bob has evidence that Alice possesses the shared secret.

---

# 8. Challenge-Response Example

Consider Alice and Bob:

```text
Alice                         Bob
  │                             │
  │                             │
  │       Random Nonce          │
  │ <────────────────────────── │
  │                             │
  │ MAC(Key, Nonce)             │
  │ ──────────────────────────> │
  │                             │
  │                             │
  │ Compare MAC values          │
  │                             │
  │ Authentication successful   │
```

The secret key itself is never transmitted.

---

# 9. Replay Attacks

Without a fresh challenge, an attacker could perform a **replay attack**.

Suppose Alice previously sent:

```text
Response = ABC123
```

An attacker called Trudi records it.

Later Trudi sends:

```text
ABC123
```

to Bob.

Bob might incorrectly believe that Alice sent the message again.

Challenge-response prevents this by using a new random nonce.

```text
Session 1:
Nonce = 12345

Session 2:
Nonce = 98765
```

The authentication response will be different for each session.

Therefore, recording the old response is not enough.

---

# 10. Man-in-the-Middle Attacks

A **Man-in-the-Middle (MITM)** attacker attempts to place themselves between two communicating parties.

```text
Alice ←────→ Trudi ←────→ Bob
```

Trudi attempts to intercept or manipulate the communication.

Strong authentication protocols use cryptographic techniques to make this significantly more difficult.

However, challenge-response must be designed carefully. Authentication alone does not automatically guarantee that every communication is protected from every type of MITM attack.

---

# 11. Accidental Errors vs. Deliberate Manipulation

Data can become corrupted for different reasons.

### Accidental corruption

For example:

```text
Network noise
Hardware failure
Transmission error
Storage corruption
```

### Deliberate manipulation

An attacker intentionally changes the data:

```text
Original:
Transfer $100

Attacker:
Transfer $1000
```

These two situations require different protection mechanisms.

---

# 12. CRC for Accidental Errors

A **Cyclic Redundancy Check (CRC)** is commonly used to detect accidental transmission errors.

Conceptually:

```text
Message
   ↓
CRC calculation
   ↓
CRC value
```

The receiver recalculates the CRC.

```text
Received Message
      ↓
Calculate CRC
      ↓
Compare
```

If the values differ, the data was probably corrupted.

CRC is excellent for detecting many accidental errors.

However:

> **CRC is not a cryptographic integrity mechanism.**

---

# 13. Why CRC Cannot Protect Against Attackers

Suppose a file contains:

```text
Transfer $100
```

and its CRC is:

```text
CRC = ABC123
```

An attacker changes the file:

```text
Transfer $1000
```

The attacker can simply calculate a new CRC:

```text
CRC = XYZ789
```

and send:

```text
Modified File + New CRC
```

The receiver sees a matching CRC.

Therefore, CRC cannot determine whether the modification was accidental or intentional.

The attacker knows how CRC works.

---

# 14. Cryptographic Hash Functions

A **cryptographic hash function** provides a stronger mechanism for detecting malicious modifications.

A hash function takes an input message of arbitrary length and produces a fixed-length output.

```text
Message
   ↓
Hash Function
   ↓
Hash / Digest
```

Mathematically:

```text
h = H(m)
```

where:

* `m` = message
* `H` = hash function
* `h` = hash/digest

For example:

```text
Message:
HELLO

       ↓ SHA-256

Digest:
[256-bit value]
```

---

# 15. Properties of Hash Functions

A secure cryptographic hash function should have several important properties.

---

## 15.1 Arbitrary Input

A hash function should be able to accept messages of different lengths.

```text
H("Hello")
H("This is a much longer message...")
H(large file)
```

All are valid inputs.

---

## 15.2 Fixed-Length Output

Regardless of input size, the hash produces an output of a fixed size.

For example:

```text
1 byte input
      ↓
256-bit hash

1 MB input
      ↓
256-bit hash

1 GB input
      ↓
256-bit hash
```

---

# 16. Fast Computation

Hash functions should be efficient to calculate.

For example:

```text
Large File
    ↓
SHA-256
    ↓
Digest
```

The hash can be calculated quickly even for large files.

This makes hashes useful for:

* File integrity checking
* Digital signatures
* Password storage systems
* Certificates
* Software verification
* Blockchain systems

---

# 17. Hashes Are Keyless

A normal hash function does not use a secret key.

```text
Message
   ↓
Hash Function
   ↓
Digest
```

This differs from a MAC:

```text
Message + Secret Key
        ↓
       MAC
```

Therefore, a hash alone does not provide authentication.

Anyone can calculate:

```text
H(message)
```

---

# 18. Hashing Is Not Encryption

A common mistake is to think:

> "Hashing is a type of encryption."

It is not.

### Encryption

```text
Plaintext + Key
      ↓
 Encryption
      ↓
Ciphertext
      ↓
 Decryption + Key
      ↓
Plaintext
```

Encryption is designed to be reversible with the appropriate key.

### Hashing

```text
Message
   ↓
Hash
```

A cryptographic hash is designed to be **one-way**.

You should not be able to recover the original message from the hash.

---

# 19. Compression and Information Loss

Hash functions can be viewed as a type of **lossy compression**.

For example:

```text
1 GB File
    ↓
256-bit Hash
```

There is no way for the 256-bit value to contain all the information necessary to reconstruct the original 1 GB file.

Therefore:

```text
Message → Hash
```

is easy to calculate, but:

```text
Hash → Original Message
```

should be computationally infeasible.

---

# 20. Avalanche Effect

A good cryptographic hash has a strong **avalanche effect**.

A tiny change in the input should produce a completely different-looking output.

For example:

```text
Input 1:
Hello

Input 2:
hello
```

Only one character changed.

Yet:

```text
H("Hello")
```

and

```text
H("hello")
```

should look completely unrelated.

Conceptually:

```text
Small input change
       ↓
Large unpredictable
output change
```

---

# 21. Random Oracle Model

The **random oracle model** is a theoretical model used to describe an ideal hash function.

An ideal random oracle behaves as though:

```text
Input
  ↓
Randomly selected output
```

The outputs should appear uniformly distributed and unpredictable.

Real hash functions are deterministic:

```text
Same Input
    ↓
Same Output
```

But a well-designed cryptographic hash should make the output appear random to an attacker.

---

# 22. Protecting a Hash

A major problem exists when using a normal hash to verify integrity.

Suppose Alice sends:

```text
Message + H(Message)
```

Trudi modifies the message.

She can simply calculate:

```text
H(Modified Message)
```

and send:

```text
Modified Message + H(Modified Message)
```

Bob has no way to know that Trudi created the new hash.

Therefore, the hash itself must be protected.

Two important solutions are:

```text
                 Hash Protection
                       │
             ┌─────────┴─────────┐
             │                   │
       Public/Private         Secret Key
             │                   │
      Digital Signature       MAC / HMAC
```

---

# 23. Message Authentication Code — MAC

A **Message Authentication Code (MAC)** uses a secret key to protect a message.

Conceptually:

```text
Message + Secret Key
        ↓
       MAC
        ↓
Authentication Tag
```

The sender sends:

```text
Message + MAC
```

The receiver uses the shared secret key to calculate the MAC again.

```text
Received Message
       +
   Secret Key
       ↓
    Calculate MAC
       ↓
   Compare values
```

If the values match, the receiver has evidence that:

1. The message was not modified.
2. The sender possessed the shared secret key.

Therefore, a MAC provides **integrity and authentication**.

---

# 24. HMAC

**HMAC (Hash-based Message Authentication Code)** is a MAC construction based on a cryptographic hash function.

Conceptually:

```text
Message + Secret Key
          ↓
         HMAC
          ↓
Authentication Tag
```

HMAC can be constructed using secure hash functions such as SHA-256.

For example:

```text
HMAC-SHA-256
```

is commonly used in cryptographic protocols and applications.

---

# 25. Digital Signatures

The other major method of protecting hashes uses **public-key cryptography**.

The sender creates a hash of the message and signs it using their private key.

```text
Message
   ↓
Hash
   ↓
Sign with Private Key
   ↓
Digital Signature
```

The receiver uses the sender's public key to verify the signature.

```text
Message
   ↓
Hash
   ↓
Compare
   ↑
Verify Signature
   ↑
Public Key
```

This provides:

* Integrity
* Authentication
* Evidence of origin
* Non-repudiation, when the surrounding system properly binds the key to the signer

---

# 26. MAC vs. Digital Signature

| Property        | MAC                  | Digital Signature                         |
| --------------- | -------------------- | ----------------------------------------- |
| Keys            | Shared secret        | Public/private key pair                   |
| Integrity       | Yes                  | Yes                                       |
| Authentication  | Yes                  | Yes                                       |
| Non-repudiation | No                   | Yes, when properly implemented            |
| Verification    | Requires secret key  | Uses public key                           |
| Typical use     | Secure communication | Contracts, certificates, software signing |

The major difference is **who can create the authentication value**.

With a MAC:

```text
Alice ─── Shared Key ─── Bob
```

Both Alice and Bob know the key.

With a digital signature:

```text
Alice:
Private Key → Sign

Bob:
Public Key → Verify
```

Only Alice should possess the private key.

---

# 27. Why Symmetric Cryptography Cannot Provide Non-Repudiation

Consider a stockbroker example.

Alice sends:

```text
"Buy 100 shares."
```

Alice and the broker share a secret key.

Alice creates:

```text
MAC(Message, SharedKey)
```

The broker verifies it successfully.

Later Alice claims:

> "I never sent that order."

The broker cannot definitively prove that Alice created the MAC.

Why?

Because the broker also knows the same secret key.

Therefore, the broker could have generated the MAC themselves.

```text
Alice ──┐
        ├── Shared Secret Key
Broker ─┘
```

Either party could potentially create a valid MAC.

Therefore, symmetric cryptography does not provide true non-repudiation.

---

# 28. Public-Key Cryptography and Non-Repudiation

Digital signatures solve this problem using asymmetric keys.

Alice has:

```text
Private Key → Secret
Public Key  → Public
```

Alice signs the message using her private key.

```text
Message
   ↓
Hash
   ↓
Private Key
   ↓
Signature
```

The broker verifies the signature using Alice's public key.

```text
Signature
    +
Alice's Public Key
    ↓
Verification
```

Assuming the private key is properly controlled and the public key is reliably bound to Alice's identity, this provides a much stronger basis for non-repudiation.

---

# 29. Hash Collision

A **collision** occurs when two different messages produce the same hash.

```text
Message A ──→ H ──→ ABC123

Message B ──→ H ──→ ABC123
```

Therefore:

```text
A ≠ B
```

but:

```text
H(A) = H(B)
```

Collisions are mathematically unavoidable because the input space is effectively unlimited while the output space is fixed.

The security requirement is that finding such collisions should be computationally infeasible.

---

# 30. Pre-Image Resistance

Suppose an attacker is given a hash:

```text
h = H(m)
```

**Pre-image resistance** means it should be computationally infeasible to find a message `m` that produces the given hash.

Conceptually:

```text
Hash
 ↓
?
 ↓
Message
```

The attacker should not be able to efficiently reverse the hash.

This is also called **one-wayness**.

---

# 31. Second Pre-Image Resistance

Suppose the attacker already has a specific message:

```text
M
```

and its hash:

```text
H(M)
```

The attacker should not be able to find another message:

```text
M' ≠ M
```

such that:

```text
H(M') = H(M)
```

This property is called **second pre-image resistance**.

---

# 32. Weak Collision Resistance

Weak collision resistance is closely related to second pre-image resistance.

The attacker is given a particular message `M`.

The goal is to find:

```text
M' ≠ M
```

such that:

```text
H(M') = H(M)
```

The attacker does not get to choose both messages freely.

---

# 33. Strong Collision Resistance

Strong collision resistance has a different goal.

The attacker can search for **any two different messages**:

```text
M ≠ M'
```

such that:

```text
H(M) = H(M')
```

The attacker is free to choose both messages.

This makes finding a collision significantly easier than finding a second pre-image.

The main reason is the **birthday paradox**.

---

# 34. The Birthday Problem

Imagine a room full of people.

You might ask:

> "How many people do I need before there is a 50% chance that someone shares my birthday?"

This requires roughly:

```text
254 people
```

But there is a different question:

> "How many people do I need before there is a 50% chance that any two people share a birthday?"

The answer is only about:

```text
23 people
```

This surprising difference is the **birthday paradox**.

---

# 35. Birthday Attacks on Hashes

The same mathematics applies to hash functions.

Suppose a hash has `n` bits.

There are:

```text
2ⁿ
```

possible hash values.

### Matching a Specific Hash

If an attacker wants to find a message matching a specific existing hash, approximately:

```text
2ⁿ
```

attempts are required in the generic case.

Therefore, a 128-bit hash has approximately:

```text
2¹²⁸
```

pre-image security.

---

# 36. Finding Any Collision

If the attacker can choose both messages, the birthday effect applies.

The attacker needs approximately:

```text
2ⁿᐟ²
```

operations to find a collision.

For a 128-bit hash:

```text
2¹²⁸
```

possible outputs

but approximately:

```text
2⁶⁴
```

work is sufficient for a generic birthday attack.

Therefore:

> An n-bit hash provides approximately n/2 bits of generic collision security.

---

# 37. Why 128-Bit Hashes Are Too Small for Strong Collision Resistance

Consider a 128-bit hash.

Its theoretical collision security is approximately:

```text
2⁶⁴
```

operations.

This is much weaker than the full 128-bit output might initially suggest.

Therefore, 128-bit hashes are generally not considered sufficient for modern strong collision resistance.

Modern applications commonly use larger hashes such as:

```text
SHA-256
SHA-384
SHA-512
```

---

# 38. Birthday Attack Example

Consider a malicious attacker called Trudi.

Suppose Trudi wants someone to sign a favorable contract.

She creates many versions:

```text
Contract A1
Contract A2
Contract A3
...
Contract A1000000
```

and also creates many unfavorable contracts:

```text
Contract B1
Contract B2
Contract B3
...
Contract B1000000
```

She searches for:

```text
H(Favorable Contract)
        =
H(Unfavorable Contract)
```

If she can find a collision, she may attempt to get the victim to sign the favorable version and later present the unfavorable version with the same hash.

This is a simplified illustration of why collision resistance is important for digital signatures and document signing.

---

# 39. Five Essential Hash Requirements

A secure cryptographic hash should provide the following properties.

## 1. Compression

Large or arbitrary-length inputs are mapped to a fixed-size output.

```text
Any-size message
       ↓
Fixed-size digest
```

---

## 2. Efficiency

The hash should be computationally efficient to calculate.

```text
Message → Hash
```

should be practical even for large messages.

---

## 3. Pre-Image Resistance

Given:

```text
H(x)
```

it should be computationally infeasible to recover `x`.

---

## 4. Weak Collision Resistance

Given `x`, it should be computationally infeasible to find:

```text
y ≠ x
```

such that:

```text
H(y) = H(x)
```

---

## 5. Strong Collision Resistance

It should be computationally infeasible to find any:

```text
w ≠ z
```

such that:

```text
H(w) = H(z)
```

---

# 40. Historical Hash Algorithms

Several important hash functions have been used historically.

Their security status has changed as cryptanalysis improved.

---

## MD4

**MD4** was introduced in:

```text
1990
```

It produces a:

```text
128-bit digest
```

Cryptanalytic weaknesses were discovered, and practical collision attacks were demonstrated.

MD4 is now completely unsuitable for security purposes.

---

# 41. MD5

**MD5** was introduced in:

```text
1992
```

It also produces:

```text
128-bit digest
```

MD5 became extremely popular because it was fast and easy to implement.

However, practical collision attacks were eventually developed.

Therefore:

> **MD5 should not be used for security-sensitive integrity, signatures, or authentication.**

It may still appear in legacy systems or non-security contexts, but it should not be selected for new cryptographic applications.

---

# 42. SHA-1

**SHA-1** was introduced in:

```text
1995
```

It produces:

```text
160-bit digest
```

SHA-1 was stronger than MD5 in several respects and was widely used in:

* Digital certificates
* Software signing
* Git
* Security protocols

However, practical collision attacks were eventually demonstrated.

Therefore, SHA-1 is no longer considered secure for collision-sensitive cryptographic applications.

---

# 43. RIPEMD-160

**RIPEMD-160** was developed as a European alternative to SHA-1.

It produces:

```text
160-bit digest
```

It was designed as a stronger successor to earlier RIPEMD variants.

RIPEMD-160 remains historically important and has been used in systems such as cryptocurrency-related applications.

However, for new general-purpose cryptographic designs, modern SHA-2 or SHA-3 family algorithms are generally preferred.

---

# 44. Hash Algorithm Timeline

The historical progression can be summarized as:

```text
1990
 │
 └── MD4
      ↓
1992
 │
 └── MD5
      ↓
1993
 │
 └── RIPEMD-160
      ↓
1995
 │
 └── SHA-1
      ↓
Modern
 │
 ├── SHA-2
 │
 └── SHA-3
```

The important lesson is:

> **Cryptographic algorithms can become insecure as new attacks and increased computing power emerge.**

---

# 45. Hash vs. MAC vs. Digital Signature

These concepts are easy to confuse.

| Feature         | Hash                                              | MAC/HMAC           | Digital Signature          |
| --------------- | ------------------------------------------------- | ------------------ | -------------------------- |
| Uses key?       | No                                                | Yes, shared secret | Yes, private key           |
| Integrity       | Yes, but not against an active attacker by itself | Yes                | Yes                        |
| Authentication  | No                                                | Yes                | Yes                        |
| Non-repudiation | No                                                | No                 | Yes*                       |
| Verification    | Anyone                                            | Secret-key holder  | Public-key holder          |
| Example         | SHA-256                                           | HMAC-SHA-256       | RSA/ECDSA/EdDSA signatures |

`*` Non-repudiation depends on proper private-key control, identity binding, and the surrounding legal/technical system.

---

# 46. Putting Everything Together

A secure communication system may combine many cryptographic mechanisms.

For example:

```text
                    Secure Communication
                           │
          ┌────────────────┼────────────────┐
          │                │                │
   Confidentiality    Authentication     Integrity
          │                │                │
      Encryption       MAC / Signature      │
          │                │                │
          └────────────────┼────────────────┘
                           │
                    Secure Protocol
                           │
                    Non-repudiation
                    when signatures
                    are appropriately used
```

A common pattern is:

```text
Message
   │
   ├──────────────► Encryption
   │
   └──────────────► Authentication Tag
                         │
                         ▼
                  Secure Transmission
```

The exact construction depends on the protocol and security requirements.

---

# 47. Key Takeaways

### Four Core Security Goals

* **Confidentiality** → Keep information secret.
* **Authentication** → Verify who sent the information.
* **Integrity** → Detect unauthorized modifications.
* **Non-repudiation** → Provide evidence of who performed an action.

### Authentication

* Shared secrets can be used to authenticate parties.
* Challenge-response protocols use fresh nonces.
* Nonces help prevent replay attacks.
* Authentication protocols must be carefully designed to resist MITM attacks.

### Integrity

* CRCs are useful for detecting accidental errors.
* CRCs are not secure against malicious attackers.
* Cryptographic hashes provide stronger integrity mechanisms.
* A plain hash alone does not authenticate the sender.

### Hash Functions

A secure hash should provide:

```text
Compression
Efficiency
Pre-image resistance
Second pre-image resistance
Collision resistance
Avalanche effect
```

### Hash Collisions

For an `n`-bit hash:

```text
Specific pre-image:
≈ 2ⁿ

Generic collision:
≈ 2ⁿᐟ²
```

This is why collision resistance is weaker than the full hash output size might suggest.

### Protection Methods

```text
Hash
 │
 ├── + Secret Key → MAC / HMAC
 │
 └── + Private Key → Digital Signature
```

### Historical Algorithms

* **MD4** → Broken
* **MD5** → Broken for collision resistance
* **SHA-1** → Broken for collision resistance
* **RIPEMD-160** → Historically important, but modern systems generally favor newer hash families

The central lesson is:

> **Encryption protects confidentiality, while authentication mechanisms, MACs, digital signatures, and cryptographic hashes provide the additional properties needed to build trustworthy modern communication systems.**
