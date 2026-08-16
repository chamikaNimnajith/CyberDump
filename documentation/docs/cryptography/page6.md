# Secret Key Assurances and Key Exchange

## 1. Secret Key Assurances

Cryptography allows two parties, such as **Alice and Bob**, to communicate securely using a shared secret key. A good cryptographic system provides several important assurances about electronic data.

The four main assurances are:

1. **Confidentiality**
2. **Authentication**
3. **Integrity**
4. **Nonrepudiation**

---

## 1.1 Confidentiality

**Confidentiality** ensures that only authorized people who possess the correct secret key can access encrypted information.

For example, imagine Alice and Bob are business partners who exchange:

* Financial information
* Personal notes
* Business documents
* Customer information

If Alice encrypts a file using their shared secret key, an attacker such as **BlackHat** may intercept the encrypted file, but should not be able to understand its contents without the key.

```text
Alice
  │
  │ Plaintext
  ▼
[ Encryption ]
  │
  │ Ciphertext
  ▼
Internet ───────────────► Bob
                              │
                              ▼
                         [ Decryption ]
                              │
                              ▼
                           Plaintext
```

Therefore:

> **Confidentiality = Only authorized parties can read the information.**

---

# 1.2 Authentication

**Authentication** provides assurance about the identity of the person or computer communicating with you.

Without authentication, an attacker could pretend to be someone else.

For example:

```text
Bob ───────────────► "Are you Alice?"
                       │
                       ▼
                    BlackHat
                       │
                       └── "Yes, I am Alice!"
```

The problem is that simply asking someone to identify themselves is not sufficient. A cryptographic system needs a way to prove that the other party actually possesses the shared secret key.

---

## Challenge and Response Protocol

A **challenge-response protocol** allows Bob to verify that Alice knows the shared secret key **without sending the secret key over the network**.

A simplified process looks like this:

```text
Bob                                  Alice

 │                                    │
 │──── Random Challenge ─────────────►│
 │                                    │
 │                              Uses secret key
 │                                    │
 │◄──── Cryptographic Response ───────│
 │                                    │
 │ Verify response using key          │
 │                                    │
 ▼                                    ▼
Authenticated                    Authenticated
```

The important idea is:

> The secret key itself is never transmitted.

Instead, Alice proves that she knows the key by correctly responding to Bob's challenge.

---

## Authentication Attacks

Challenge-response authentication can still be vulnerable if poorly designed.

### Replay Attack

Suppose BlackHat is monitoring the communication between Alice and Bob.

BlackHat records:

```text
Challenge → Response
```

Later, BlackHat sends the same response again.

If Bob accepts the old response, BlackHat may successfully impersonate Alice.

```text
First communication:

Bob ── Challenge A ──► Alice
Bob ◄── Response A ─── Alice

BlackHat records both.

Later:

Bob ◄── Response A ─── BlackHat
```

This is known as a **replay attack**.

---

## Randomization

To prevent replay attacks, the challenge should be different each time.

A challenge can be generated randomly from a very large range:

```text
Challenge 1 → 847291...
Challenge 2 → 193847...
Challenge 3 → 729104...
```

If the challenge is unpredictable and rarely repeated, an attacker cannot simply reuse an old response.

Therefore:

> **Random challenges help prevent replay attacks.**

---

## Pseudo-Random Numbers

Computers are deterministic machines, so generating truly random numbers is difficult.

Instead, computers commonly generate **pseudo-random numbers** using algorithms.

A pseudo-random sequence might look random:

```text
17 → 82 → 43 → 91 → 26 → 74 → ...
```

However, if the underlying algorithm or its starting value can be predicted, an attacker may discover patterns and predict future values.

This creates a serious security problem.

```text
Predictable challenge
        ↓
Attacker predicts next challenge
        ↓
Attacker prepares response
        ↓
Authentication bypass
```

Cryptographic systems therefore require **cryptographically secure random number generation**.

---

## One-Time Pad

The lecture also introduces the historical concept of the **one-time pad**.

A one-time pad uses a truly random key that:

* Is at least as long as the message
* Is used only once
* Is kept completely secret
* Is never reused

When these conditions are satisfied, the one-time pad provides theoretically perfect secrecy.

The important connection is the use of **random, non-repeating keys**.

---

# 1.3 Integrity

**Integrity** provides assurance that a message or file has not been modified during transmission.

For example, Alice sends:

```text
Transfer $1000 to Bob
```

An attacker might try to change it to:

```text
Transfer $9000 to BlackHat
```

Encryption alone does not necessarily provide a way to detect every type of modification.

Integrity mechanisms allow Bob to determine whether the message was altered.

---

## Message Authentication Codes (MAC)

A **Message Authentication Code (MAC)** is a cryptographic value calculated using:

* The message
* A shared secret key

Conceptually:

```text
Message + Secret Key
          │
          ▼
       MAC Algorithm
          │
          ▼
     MAC / Fingerprint
```

Alice sends:

```text
Message + MAC
```

Bob receives the message and MAC and independently calculates the MAC using his copy of the secret key.

```text
Received Message
       +
   Secret Key
       │
       ▼
   MAC Algorithm
       │
       ▼
Calculated MAC
```

Bob then compares the calculated MAC with the received MAC.

### If they match

```text
Received MAC = Calculated MAC
        ↓
Message is considered authentic and unmodified
```

### If they do not match

```text
Received MAC ≠ Calculated MAC
        ↓
Message may have been modified
```

---

## Why Small Changes Are Detectable

Cryptographic algorithms are designed so that a small change in the input produces a significantly different output.

For example:

```text
Original:
"Transfer $1000"

Modified:
"Transfer $9000"
```

Even a tiny change can result in a completely different MAC.

This property makes it difficult for an attacker to modify a message while keeping the same valid MAC.

---

# MACs for Plaintext Messages

A MAC can be useful even when confidentiality is not required.

Consider a **public software update**.

The software itself does not need to be secret. Anyone may download it.

However, users still need to know:

> "Did this software actually come from the legitimate publisher, and was it modified?"

A MAC or another integrity/authentication mechanism can help provide this assurance.

There is also a practical advantage: calculating a MAC can be much less computationally expensive than encrypting and decrypting large amounts of data when confidentiality is unnecessary.

---

# MACs as Compressed Fingerprints

A MAC can be thought of as a small cryptographic fingerprint of a potentially very large file.

For example:

```text
Large File
   │
   ▼
MAC Algorithm + Secret Key
   │
   ▼
Small MAC
```

A 1 MB file and a much larger file can both produce a short MAC value.

The MAC does **not** contain a compressed copy of the original file.

Instead, it is a one-way cryptographic value used for verification.

Therefore:

> A MAC can verify a message without providing a way to reconstruct the original message from the MAC.

---

# 1.4 Nonrepudiation

**Nonrepudiation** provides assurance that a sender cannot later falsely deny sending a message or file.

For example:

```text
Alice ───────► Contract
```

Later, Alice should not be able to falsely claim:

> "I never sent that contract."

However, **shared secret keys alone cannot provide strong nonrepudiation**.

Why?

Because both Alice and Bob possess the same secret key.

If a MAC was generated using that key, Bob could potentially have generated the same MAC himself.

Therefore, Bob cannot use the shared-key MAC as definitive proof that Alice was the sender.

Nonrepudiation generally requires:

* **Public-key cryptography**
* **Digital signatures**
* Or an appropriate **trusted third party**

---

# 2. Problems With Secret Key Exchange

Secret-key cryptography has an important practical problem:

> **How do Alice and Bob securely obtain the same secret key in the first place?**

If Alice sends the key over an insecure network:

```text
Alice ───── Secret Key ─────► Bob
                  │
                  ▼
               BlackHat
```

BlackHat could intercept the key.

Once the attacker has the key, encrypted communications protected by that key may no longer be secure.

This is known as the **key distribution problem**.

---

# 2.1 The Ancient Problem of Secret Information

The difficulty of protecting secret information is not new.

The lecture references **Sun-Tzu's *The Art of War* (*Ping-fa*)**, highlighting the importance of obtaining, protecting, and securely communicating intelligence.

The fundamental problem remains the same:

```text
Secret information
       ↓
Must be delivered
       ↓
Must be stored securely
       ↓
Must be protected from opponents
```

Modern cryptography solves many aspects of this problem, but secure key distribution remains an important challenge.

---

# 2.2 Key Growth and Scalability

Secret-key systems become increasingly difficult to manage as the number of users increases.

If every pair of users needs a unique shared key, the number of keys required is:

[
\frac{N(N-1)}{2}
]

where **N** is the number of parties.

### Examples

| Number of Parties | Required Shared Keys |
| ----------------: | -------------------: |
|                10 |                   45 |
|               100 |                4,950 |
|             1,000 |              499,500 |

The problem grows very quickly.

For example:

```text
10 users
   ↓
45 keys

100 users
   ↓
4,950 keys

1,000 users
   ↓
499,500 keys
```

This makes large-scale secret-key management extremely difficult.

---

# 2.3 Trusted Third Party (TTP)

One solution is to introduce a **Trusted Third Party (TTP)**.

Instead of every pair of users independently exchanging keys, a trusted organization or system manages the keys.

A common example is a:

> **Key Distribution Center (KDC)**

Conceptually:

```text
          ┌─────────────┐
          │     KDC     │
          │ Trusted     │
          │ Third Party │
          └──────┬──────┘
                 │
        ┌────────┼────────┐
        ▼        ▼        ▼
      Alice     Bob     Charlie
```

The KDC can help users obtain temporary or session keys for secure communication.

---

# 2.4 The Military Model

A traditional model places a trusted authority between communicating parties.

For example:

```text
Alice ──► TTP ──► Bob
```

The trusted authority may participate in establishing or protecting communications.

This can create additional work because information or keys may need to pass through the trusted authority.

In some designs, the TTP may even encrypt and decrypt messages, effectively becoming an intermediary.

---

# 2.5 Encrypting Keys with Other Keys

Cryptographic keys can themselves be protected using another cryptographic key.

This is commonly called **key wrapping** or **key encryption**.

Conceptually:

```text
Secret Key
    │
    ▼
Encryption Key
    │
    ▼
Encrypted / Wrapped Key
```

The encrypted key can then be stored or transported more safely.

However, the mechanism used to protect a key must itself provide an appropriate level of security.

---

# 2.6 Trust Is a Security Problem

A TTP is only useful if it can actually be trusted.

The lecture illustrates this idea using the historical example of **Axel Fersen**, who acted as a trusted intermediary for **King Louis XVI and Marie Antoinette**.

The broader lesson is:

> A trusted third party introduces another entity that must itself be trusted and protected.

If the TTP is compromised, dishonest, or careless, the security of the entire system may be affected.

---

# 2.7 Problems With KDCs and TTPs

Although a KDC can simplify key management, it introduces several problems.

### 1. Key Management Logistics

Large organizations may have thousands or millions of users.

Managing, distributing, replacing, and revoking secret keys becomes difficult.

---

### 2. Single Point of Compromise

A KDC may contain or manage a large number of secret keys.

Therefore, it becomes a highly attractive target.

```text
              KDC
        ┌─────────────┐
        │ Many Keys   │
        │ User Data   │
        │ Credentials │
        └─────────────┘
               ▲
               │
        Attractive Target
```

If an attacker compromises the KDC, many users may be affected.

---

### 3. Previously Captured Ciphertext

Suppose BlackHat records encrypted communications today:

```text
Encrypted Message 1
Encrypted Message 2
Encrypted Message 3
```

The attacker cannot decrypt them immediately.

However, if the corresponding secret key is stolen later:

```text
Secret Key Compromised
        ↓
Old encrypted messages
        ↓
May be decrypted
```

Therefore, protecting keys over their entire lifetime is extremely important.

---

### 4. Secret Keys Have Limited Lifetimes

A secret key can eventually become compromised because of:

* Theft
* Unauthorized copying
* Poor storage
* Accidental disclosure
* Malware
* Insider threats
* Weak key management

Therefore, keys should not necessarily be treated as permanently secure.

Organizations often use **key rotation**, where keys are periodically replaced.

---

# 3. Public-Key Cryptography as a Solution

The problems associated with distributing and managing shared secret keys helped motivate the development and adoption of **public-key cryptography**.

With symmetric cryptography:

```text
Alice ── Shared Secret Key ── Bob
```

The main challenge is securely getting that shared secret key to both parties.

With public-key cryptography, each user has a **key pair**:

```text
Alice:
  Public Key
  Private Key

Bob:
  Public Key
  Private Key
```

Public keys can be distributed openly, while private keys remain secret.

This greatly reduces the key-distribution problem and enables important security technologies such as:

* Digital signatures
* Authentication
* Nonrepudiation
* Secure key exchange

---

