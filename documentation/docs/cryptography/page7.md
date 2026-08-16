# Key Exchange, Diffie–Hellman, and RSA

## 1. The Key Exchange & Key Distribution Problem

One of the biggest challenges in symmetric cryptography is **securely distributing the secret key**.

Symmetric encryption requires both parties to know the same secret key:

```text
Alice                         Bob
  │                            │
  │      Shared Secret Key     │
  └────────────────────────────┘
```

The problem is:

> **How can Alice securely give the secret key to Bob if their communication channel is not secure?**

An attacker such as **Trudy** may be monitoring the network.

```text
Alice ─────── Secret Key ───────► Bob
                    │
                    ▼
                  Trudy
              (eavesdropper)
```

If Trudy obtains the key, she can potentially decrypt the communications.

### Physical Key Distribution

For a small organization, keys could theoretically be delivered using:

* A trusted person
* A secure courier
* Physical media
* Face-to-face meetings

However, this becomes extremely expensive and impractical when thousands or millions of users need secure communication.

---

# 1.1 Key Distribution Centers

A **Key Distribution Center (KDC)** is a trusted central system that helps users establish shared keys.

Without a KDC, every pair of users needs its own secret key.

For example:

```text
Alice ←──── Key ────→ Bob
Alice ←──── Key ────→ Charlie
Bob   ←──── Key ────→ Charlie
```

As the number of users increases, the number of required keys grows rapidly.

A KDC changes the model.

Each user only needs a key shared with the KDC:

```text
          KDC
        /  |  \
       /   |   \
   Alice  Bob  Charlie
```

For example:

```text
Alice ── Key A ── KDC
Bob   ── Key B ── KDC
Charlie ─ Key C ─ KDC
```

The KDC can then help Alice and Bob establish a temporary session key.

### Advantages

* Reduces the number of keys each user must manage
* Centralizes key management
* Makes large systems easier to administer

### Problems

The KDC itself becomes a critical security component.

A large KDC may manage a very large number of keys, making it an attractive target for attackers.

It can also become:

* A **single point of compromise**
* A **performance bottleneck**
* A **scalability limitation**

This becomes particularly important in large-scale systems such as modern e-commerce platforms.

---

# 2. Merkle's Puzzles

Before modern public-key cryptography was widely developed, **Ralph Merkle** proposed an interesting method for allowing two parties to establish a secret over a public channel.

This idea became one of the important foundations of public-key cryptography.

The basic idea is to make the problem:

> **Easy for the legitimate receiver but much harder for an eavesdropper.**

---

## 2.1 How Merkle's Puzzles Work

Suppose Alice wants to establish a secret key with Bob.

Alice creates a very large collection of puzzles.

Each puzzle contains:

* A secret key
* A serial number
* Weak encryption

For example:

```text
Puzzle 1 → Serial 001 → Weakly encrypted key
Puzzle 2 → Serial 002 → Weakly encrypted key
Puzzle 3 → Serial 003 → Weakly encrypted key
...
Puzzle N → Serial N   → Weakly encrypted key
```

Alice sends the entire collection to Bob over a public communication channel.

Trudy can also intercept the collection.

---

## 2.2 Bob's Advantage

Bob randomly selects one puzzle.

He solves it, obtains the secret key, and learns its serial number.

For example:

```text
Bob chooses Puzzle 573821
          ↓
Solves puzzle
          ↓
Obtains Key X
          ↓
Sends serial number 573821 to Alice
```

Alice can identify the same puzzle and therefore knows that they should use **Key X**.

---

## 2.3 Trudy's Problem

Trudy has the entire collection and also sees the serial number.

However, she does not know which puzzle contains the correct key.

She may have to solve many puzzles until she finds the correct one.

If there are **1,000,000 puzzles**, Trudy may need to solve approximately half of them on average.

```text
1,000,000 puzzles
       ↓
Average search
       ↓
~500,000 puzzles
```

This creates a significant difference in computational effort.

The legitimate participants solve approximately one puzzle, while the attacker may need to solve hundreds of thousands.

The important idea is:

> **The legitimate user gets a computational advantage over the attacker.**

Merkle's Puzzles were an important conceptual step toward public-key cryptography.

---

# 3. Diffie–Hellman Key Exchange

The **Diffie–Hellman key exchange**, introduced publicly in 1976 by **Whitfield Diffie and Martin Hellman**, provided a major breakthrough.

It allows two parties to establish a shared secret over a public communication channel without directly transmitting the secret itself.

The underlying mathematics had also been independently developed in classified work by **Malcolm Williamson at GCHQ**.

---

# 3.1 Basic Idea

Alice and Bob publicly agree on two values:

* A large prime number **p**
* A base **g**

These values do not need to be secret.

```text
Public:
p = large prime
g = base
```

Alice chooses a private number:

```text
a = Alice's private secret
```

Bob chooses:

```text
b = Bob's private secret
```

The private values **a** and **b** are never transmitted.

---

# 3.2 Alice's Calculation

Alice calculates:

[
A = g^a \mod p
]

She sends **A** to Bob.

---

# 3.3 Bob's Calculation

Bob calculates:

[
B = g^b \mod p
]

He sends **B** to Alice.

The communication looks like:

```text
Alice                                  Bob

Private: a                             Private: b

A = g^a mod p                          B = g^b mod p

       ───────── A ─────────────────►
       ◄───────── B ─────────────────
```

An attacker can see:

* p
* g
* A
* B

But the attacker should not be able to efficiently determine **a** or **b**.

---

# 3.3 Creating the Shared Secret

Alice uses Bob's public value:

[
K = B^a \mod p
]

Substituting:

[
K = (g^b)^a \mod p
]

Therefore:

[
K = g^{ab} \mod p
]

Bob performs:

[
K = A^b \mod p
]

Therefore:

[
K = (g^a)^b \mod p
]

So both obtain:

[
\boxed{K = g^{ab} \mod p}
]

They now have the same shared secret.

```text
Alice                              Bob

g^a mod p  ─────────────────────►
           ◄───────────────────── g^b mod p

       Both calculate g^(ab) mod p

              ↓
       Shared Secret Key
```

The key itself was **never transmitted across the network**.

---

# 3.4 The Discrete Logarithm Problem

The security of traditional Diffie–Hellman relies on the difficulty of the **discrete logarithm problem**.

Suppose an attacker knows:

[
g^a \mod p
]

The attacker needs to determine **a**.

In a sufficiently large and properly chosen group, this is computationally difficult.

Conceptually:

```text
Easy:

a → g^a mod p

Very difficult:

g^a mod p → a
```

This is an example of a **one-way mathematical problem**.

---

# 3.5 Man-in-the-Middle Attack

Basic Diffie–Hellman has an important weakness:

> **It does not inherently authenticate the communicating parties.**

An attacker called Trudy can exploit this.

Instead of simply listening, Trudy actively intercepts and replaces the exchanged public values.

```text
Alice              Trudy               Bob

   A ─────────────► │
                    │ ─────────────► A'
                    │
   B' ◄─────────────│
                    │ ◄───────────── B
```

Trudy establishes one shared secret with Alice and another shared secret with Bob.

The result can be:

```text
Alice ←── Key 1 ──→ Trudy ←── Key 2 ──→ Bob
```

Trudy can then intercept, decrypt, modify, and re-encrypt messages.

This is called a:

> **Man-in-the-Middle (MitM) attack**

---

## Preventing MitM Attacks

Diffie–Hellman must be combined with **authentication** to prevent this problem.

Common approaches include:

* Digital signatures
* Certificates
* Authenticated key exchange
* Public-key authentication

Therefore:

> **Key exchange provides a shared secret, while authentication verifies who is participating in the exchange.**

---

# 4. Foundations of Asymmetric Cryptography

Symmetric cryptography uses the same secret key for encryption and decryption.

Asymmetric cryptography uses **two mathematically related keys**:

1. **Public key**
2. **Private key**

```text
              User
               │
        ┌──────┴──────┐
        │             │
   Public Key     Private Key
   Share openly   Keep secret
```

The public key can be distributed to everyone.

The private key must remain secret.

---

# 4.1 Why Asymmetric Cryptography Is Different

With symmetric cryptography:

```text
Encryption Key = Decryption Key
```

With asymmetric cryptography:

```text
Encryption Key ≠ Decryption Key
```

The keys are mathematically related, but knowing the public key should not allow an attacker to practically calculate the private key.

This solves an important part of the key distribution problem.

---

# 4.2 Trapdoor One-Way Functions

Asymmetric cryptography relies on mathematical problems that are:

> **Easy to perform in one direction but extremely difficult to reverse without special information.**

This is known as a **trapdoor one-way function**.

Conceptually:

```text
Input
  │
  ▼
Mathematical Function
  │
  ▼
Output
```

Calculating the output is easy.

However:

```text
Output
  │
  ▼
Trying to recover input
  │
  ▼
Extremely difficult
```

The **trapdoor** is special information—typically related to the private key—that makes the reverse operation practical.

---

# 4.3 Examples of Public-Key Cryptosystems

Several mathematical approaches have been used to build public-key cryptosystems.

| Cryptosystem                | Mathematical Foundation    |
| --------------------------- | -------------------------- |
| RSA                         | Integer factorization      |
| ElGamal                     | Discrete logarithms        |
| Diffie–Hellman              | Discrete logarithms        |
| NTRU                        | Lattice-based mathematics  |
| McEliece                    | Error-correcting codes     |
| Elliptic-curve cryptography | Elliptic-curve mathematics |
| Knapsack                    | Knapsack problems          |

**Knapsack cryptography** is historically important but the original cryptosystem was broken and is not considered secure.

---

# 5. RSA Cryptosystem

**RSA** is one of the most famous public-key cryptosystems.

It was developed in 1977 by:

* **Ronald Rivest**
* **Adi Shamir**
* **Leonard Adleman**

The name **RSA** comes from their surnames.

A related classified system had previously been independently developed by **Clifford Cocks at GCHQ in 1973**.

RSA's security is primarily associated with the difficulty of **factoring large composite integers**.

---

# 5.1 RSA Key Generation

RSA key generation involves several mathematical steps.

### Step 1: Choose Two Large Prime Numbers

Choose two large primes:

[
p
]

and

[
q
]

For real RSA systems, these primes are extremely large.

---

### Step 2: Calculate the Modulus

Calculate:

[
N = p \times q
]

The value **N** becomes part of both the public and private keys.

---

### Step 3: Calculate Euler's Totient

Calculate:

[
\phi(N) = (p-1)(q-1)
]

This value is used to construct the key pair.

---

### Step 4: Select the Public Exponent

Choose a value **e** such that:

[
gcd(e,\phi(N)) = 1
]

In other words, **e** must have no common factor with (\phi(N)) other than 1.

---

### Step 5: Calculate the Private Exponent

The private exponent **d** is calculated so that:

[
e \times d \equiv 1 \pmod{\phi(N)}
]

The **Extended Euclidean Algorithm** is commonly used to calculate **d**.

---

# 5.2 RSA Keys

The resulting keys are:

### Public Key

[
\boxed{(e,N)}
]

The public key can be distributed openly.

### Private Key

[
\boxed{(d,N)}
]

The private key must be kept secret.

```text
Public Key  → (e, N)
                 │
                 └── Can be shared

Private Key → (d, N)
                 │
                 └── Must remain secret
```

---

# 5.3 RSA Encryption

Suppose a plaintext message block is represented by:

[
m
]

RSA encryption calculates:

[
\boxed{c = m^e \mod N}
]

where:

* (m) = plaintext
* (e) = public exponent
* (N) = modulus
* (c) = ciphertext

Anyone who has Bob's public key can encrypt a message for Bob.

```text
Alice
  │
  │ Message
  ▼
Bob's Public Key
  │
  ▼
RSA Encryption
  │
  ▼
Ciphertext
  │
  └──────────────► Bob
```

---

# 5.4 RSA Decryption

Bob uses his private exponent **d**:

[
\boxed{m = c^d \mod N}
]

Therefore:

```text
Ciphertext
    │
    ▼
Bob's Private Key
    │
    ▼
RSA Decryption
    │
    ▼
Plaintext
```

The security depends on keeping the private key secret and choosing sufficiently large parameters.

---

# 5.5 Generating RSA Primes

RSA requires large prime numbers.

A simplified process is:

```text
Random Number
      │
      ▼
Primality Test
      │
 ┌────┴────┐
 │         │
Prime    Not Prime
 │         │
 ▼         └──► Try another number
Use it
```

Random candidate numbers can be generated and tested using appropriate primality-testing algorithms.

Historical descriptions may mention methods such as **Fermat-based probable-prime tests** and sieving techniques.

Modern implementations use stronger and carefully designed primality tests.

---

# 5.6 RSA Exponents

The public exponent **e** should be chosen so that it is mathematically compatible with (\phi(N)).

Historically, small values such as:

[
e=3
]

were sometimes used.

Another common choice is:

[
e=2^{16}+1=65537
]

The value **65537** is widely used because it provides efficient public-key operations while avoiding some of the problems associated with very small exponents.

Using **e = 3** without appropriate modern padding is unsafe and is generally discouraged.

---

# 5.7 Private Operations Are More Expensive

RSA uses the private exponent **d** for decryption and signing.

The private exponent is generally large.

Therefore:

```text
Public-key operation
        ↓
Usually faster

Private-key operation
        ↓
Usually slower
```

This is one reason RSA is not normally used to encrypt large amounts of data directly.

---

# 5.8 RSA Padding

Basic RSA is **deterministic**.

This means that encrypting the same plaintext with the same public key can produce the same ciphertext.

For example:

```text
"YES" → Ciphertext A
"YES" → Ciphertext A
```

This creates security problems.

An attacker could create a dictionary of possible plaintexts and their corresponding ciphertexts.

Therefore, RSA must use secure **randomized padding schemes**.

Examples include standards from the **PKCS** family and modern schemes such as **RSA-OAEP** for encryption.

The important principle is:

> **Never use textbook RSA directly for real-world encryption.**

---

# 5.9 RSA Cryptanalysis

Suppose an attacker knows the public key:

[
(e,N)
]

The attacker wants to obtain the private key.

One important attack strategy is attempting to factor:

[
N=pq
]

If the attacker can determine **p** and **q**, they can calculate:

[
\phi(N)=(p-1)(q-1)
]

and potentially recover the private exponent **d**.

Therefore:

```text
Large N
  ↓
Difficult to factor
  ↓
Private key remains protected
```

However:

```text
Small / weak N
  ↓
Factorization becomes practical
  ↓
RSA security fails
```

Modern RSA therefore requires sufficiently large and properly generated parameters.

---

# 6. Symmetric vs Asymmetric Cryptography

Symmetric and asymmetric cryptography have different strengths.

| Feature            | Symmetric Cryptography | Asymmetric Cryptography |
| ------------------ | ---------------------- | ----------------------- |
| Keys               | Shared secret          | Public + private        |
| Speed              | Very fast              | Relatively slow         |
| Bulk encryption    | Excellent              | Generally unsuitable    |
| Key distribution   | Difficult              | Easier                  |
| Digital signatures | No                     | Yes                     |
| Nonrepudiation     | Not by itself          | Supports it             |
| Examples           | AES, ChaCha20          | RSA, ECC                |

---

# 6.1 Key-Length Comparisons

Key sizes should **not** be compared directly between symmetric and asymmetric algorithms.

For example:

```text
128-bit symmetric key
        ≠
512-bit RSA key
```

The security of an algorithm depends on its underlying mathematical problem and attack complexity, not simply on the number of bits in the key.

A relatively short symmetric key can provide security comparable to a much larger RSA key.

Also, **512-bit RSA is obsolete and insecure today**.

---

# 7. Applications of Public-Key Cryptography

Public-key cryptography is powerful but computationally expensive.

Therefore, it is generally **not used to encrypt large files or continuous streams of data**.

Instead, it is commonly used for:

### 1. Session Key Distribution

Public-key cryptography can securely establish or transport a temporary symmetric key.

### 2. Digital Signatures

A private key can be used to create a digital signature that others can verify using the corresponding public key.

### 3. Authentication

Public-key mechanisms can help prove the identity of users and systems.

### 4. Secure Key Exchange

Protocols such as Diffie–Hellman allow parties to establish shared secrets without directly transmitting them.

---

# 8. Hybrid Encryption

A major practical application combines the strengths of both cryptographic approaches.

This is called **hybrid encryption**.

The basic idea is:

> **Use asymmetric cryptography to protect a symmetric session key, then use the symmetric key to encrypt the actual data.**

---

## 8.1 Example

Suppose Alice wants to communicate securely with Bob.

### Step 1 — Bob Provides His Public Key

Bob's public key is available to Alice.

```text
Bob
 │
 └── Public Key ──► Alice
```

### Step 2 — Alice Generates a Temporary Session Key

Alice generates a random symmetric key:

```text
Session Key = K
```

### Step 3 — Alice Encrypts the Session Key

Alice encrypts **K** using Bob's public key:

```text
K + Bob's Public Key
        │
        ▼
Asymmetric Encryption
        │
        ▼
Encrypted Session Key
```

She sends it to Bob.

### Step 4 — Bob Decrypts the Session Key

Bob uses his private key:

```text
Encrypted Session Key
        │
        ▼
Bob's Private Key
        │
        ▼
Session Key K
```

Now both parties know the same symmetric session key.

### Step 5 — Use Symmetric Encryption for Data

Alice and Bob can now use the fast symmetric key to protect their actual communication.

```text
             Public-Key Cryptography
                    │
                    ▼
             Secure Session Key
                    │
                    ▼
          Symmetric Cryptography
                    │
                    ▼
             Bulk Data Encryption
```

This approach combines:

* **Asymmetric cryptography** → secure key establishment
* **Symmetric cryptography** → fast data encryption

This general principle is used in modern secure communication protocols, including systems such as **TLS/HTTPS**.


