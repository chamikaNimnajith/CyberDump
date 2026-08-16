# Password Security, System Attacks, Authentication, and Session Keys

This chapter covers four important areas of cryptographic security:

1. Passwords and Password Security
2. System and Hardware Attacks
3. Network Authentication Protocols
4. Session Keys and Key Agreement

These topics demonstrate an important principle:

> **Strong cryptographic algorithms alone are not enough. The way keys, passwords, hardware, and authentication protocols are implemented is equally important.**

---

# 1. Passwords and Password Security

Passwords are one of the most common methods of authentication.

A password is essentially a secret chosen by a user and used to prove their identity.

However, passwords are very different from cryptographic keys.

---

## 1.1 Password Space vs. Key Space

A cryptographic key is normally generated randomly.

For example, a 128-bit key has:

[
2^{128}
]

possible values.

This is an enormous search space.

Passwords are usually much weaker because humans choose them.

For example, suppose a password contains 8 characters.

If we assume approximately 62 possible characters:

* 26 lowercase letters
* 26 uppercase letters
* 10 digits

then the theoretical password space is:

[
62^8
]

which is approximately:

[
2^{47.6}
]

So an 8-character password provides roughly **48 bits of theoretical entropy** under this simplified assumption.

Compare:

```text
8-character password
≈ 48 bits

128-bit cryptographic key
= 128 bits
```

The difference is enormous.

```text
Password
   ↓
Small search space
   ↓
Easier to guess

Random cryptographic key
   ↓
Huge search space
   ↓
Much harder to brute-force
```

The actual security of a password depends on how unpredictable it is, not simply its length.

---

# 1.2 Brute-Force Attacks

A **brute-force attack** tries possible passwords until the correct one is found.

For example:

```text
123456
123457
123458
123459
...
```

If the password is selected from a small space, brute force can be practical.

The attacker attempts:

[
Password_1
\rightarrow Password_2
\rightarrow Password_3
\rightarrow \cdots
]

until a correct password is discovered.

The larger and more unpredictable the password space, the more difficult brute force becomes.

---

# 1.3 Dictionary Attacks

A pure brute-force attack may test every possible combination.

A **dictionary attack** is more intelligent.

Attackers know that humans tend to choose memorable passwords.

For example:

```text
password
123456
qwerty
welcome
admin
letmein
password123
```

Instead of trying billions of random combinations first, an attacker can try a list of likely passwords.

```text
Common Password List
        │
        ▼
   Test Passwords
        │
        ▼
   Find Match
```

This is extremely effective against weak human-generated passwords.

---

# 1.4 Password Verification

A secure system should **not store passwords in plaintext**.

Bad approach:

```text
Database

Alice → MyPassword123
Bob   → Summer2026
```

If the database is stolen, the attacker immediately obtains everyone's passwords.

Instead, the system stores a password-derived value.

```text
Password
   │
   ▼
Password Hashing Function
   │
   ▼
Stored Hash
```

When the user logs in:

```text
Entered Password
      │
      ▼
Same Password Hashing Process
      │
      ▼
New Hash
      │
      ▼
Compare with Stored Hash
```

If the values match, authentication succeeds.

---

# 1.5 Hashing as Key Derivation

A password is not normally suitable for direct use as a cryptographic key.

Password hashing or **key derivation functions (KDFs)** transform a password into a value suitable for verification or cryptographic use.

The basic idea is:

[
Password \rightarrow KDF \rightarrow Derived\ Value
]

Modern password-storage systems should use password-specific KDFs such as:

* Argon2
* scrypt
* bcrypt
* PBKDF2

These algorithms are deliberately designed to make password guessing expensive.

---

# 1.6 The Problem With Simple Hashing

Suppose a website stores:

[
SHA256(password)
]

An attacker who steals the database can calculate hashes for millions of common passwords.

```text
"password" → SHA-256 → Hash A
"123456"   → SHA-256 → Hash B
"qwerty"   → SHA-256 → Hash C
```

The attacker compares these values with the stolen database.

Modern CPUs and GPUs can calculate ordinary hashes extremely quickly.

Therefore:

> **A fast general-purpose hash function is not sufficient for secure password storage.**

---

# 1.7 Salt

A **salt** is a random value added to a password before password hashing.

For example:

```text
Password + Random Salt
          │
          ▼
       KDF / Hash
          │
          ▼
      Stored Hash
```

Suppose two users choose the same password:

```text
Alice → "password"
Bob   → "password"
```

Without a salt:

```text
Alice → Same Hash
Bob   → Same Hash
```

With different salts:

```text
Alice:
Password + Salt A → Hash A

Bob:
Password + Salt B → Hash B
```

The resulting values are different.

---

# 1.8 Why Salt Helps

Salts make **pre-computed password tables** much less useful.

Without salts, an attacker could create a large table:

```text
password → hash
123456   → hash
qwerty   → hash
admin    → hash
```

This is sometimes associated with **rainbow tables** and other pre-computation techniques.

With a unique random salt for every password, the attacker has to perform the password-guessing computation separately for each salt.

Therefore:

> **A salt does not make a weak password strong, but it makes large-scale pre-computation much less effective.**

The salt does not need to be secret. It is normally stored alongside the password hash.

---

# 1.9 Iterated Hashing

Another defense is to deliberately make password processing expensive.

Instead of:

[
H(password)
]

the system may repeatedly process the password:

[
H(H(H(...H(password))))
]

for many iterations.

Conceptually:

```text
Password
   ↓
Hash
   ↓
Hash
   ↓
Hash
   ↓
Hash
   ↓
...
   ↓
Final Derived Value
```

This slows down every password guess.

For example:

```text
Without KDF:
1 guess → very fast

With expensive KDF:
1 guess → significantly slower
```

If an attacker needs to test millions of passwords, even a small increase in the cost of each guess becomes significant.

Modern password KDFs combine techniques such as:

* Salting
* Iteration
* CPU cost
* Memory cost
* Parallelism controls

---

# 1.10 Unix Password Storage

Traditional Unix systems historically stored password information in:

```text
/etc/passwd
```

Early Unix password systems used a password hashing mechanism based on the **`crypt`** function and a small salt.

The traditional salt was publicly accessible and was not intended to be secret.

A major problem was that `/etc/passwd` needed to be readable for many system functions.

This led to a more secure design.

---

# 1.11 `/etc/shadow`

Modern Unix/Linux systems generally store password hashes in:

```text
/etc/shadow
```

The shadow file has much more restricted permissions than `/etc/passwd`.

A simplified entry may look like:

```text
username:$algorithm$salt$hash:...
```

The exact format depends on the hashing scheme.

Modern Linux systems can use stronger password-hashing algorithms such as:

* yescrypt
* SHA-512 based schemes
* bcrypt on supported systems
* other PAM-supported password KDFs

Older systems may have used MD5-based password hashes.

The important lesson is:

> **Password storage has evolved from simple, relatively weak schemes toward salted and computationally expensive password hashing.**

---

# 2. System and Hardware Attacks

Cryptographic security can be attacked without directly breaking the underlying mathematics.

An attacker may instead target:

* The implementation
* The operating system
* Hardware
* Memory
* Timing
* Power consumption
* Electromagnetic radiation

These are commonly called **side-channel attacks** when they exploit information leaked by the implementation.

---

# 2.1 Classic Cryptographic Attacks

Some attacks target the communication protocol rather than the cryptographic algorithm itself.

Important examples include:

### Man-in-the-Middle

An attacker intercepts and modifies communication between two parties.

```text
Alice ←──► Trudy ←──► Bob
```

### Replay Attack

An attacker records a valid message and sends it again later.

```text
Valid Message
     ↓
Recorded
     ↓
Replayed Later
```

### Dictionary Attack

An attacker tries likely plaintexts or passwords from a prepared list.

These attacks demonstrate that:

> **A mathematically strong algorithm can still be used insecurely.**

---

# 2.2 Side-Channel Attacks

A **side-channel attack** does not necessarily attack the mathematical algorithm directly.

Instead, it observes unintended information produced during computation.

Examples include:

* Execution time
* CPU cache behavior
* Branch prediction behavior
* Power consumption
* Electromagnetic emissions
* Fault behavior

Conceptually:

```text
Secret Key
    │
    ▼
Cryptographic Operation
    │
    ├── Output
    ├── Timing
    ├── Power
    ├── Cache Activity
    └── Electromagnetic Emissions
```

The normal output may reveal nothing about the key.

However, the side-channel information may leak enough information for an attacker to recover it.

---

# 2.3 Timing Attacks

Suppose a cryptographic operation takes different amounts of time depending on the secret key.

An attacker repeatedly measures the execution time.

```text
Operation 1 → 10.2 ms
Operation 2 → 10.7 ms
Operation 3 → 10.2 ms
Operation 4 → 11.1 ms
```

With enough measurements and careful statistical analysis, the attacker may infer information about the secret.

This is known as a **timing attack**.

---

# 2.4 RSA Timing Attacks

RSA implementations often perform modular exponentiation.

A common method is **repeated squaring**.

Conceptually:

```text
Private Exponent d
       │
       ▼
Repeated Squaring
       │
       ▼
Modular Multiplication
```

If the implementation performs different operations depending on whether a particular bit of the private exponent is 0 or 1, execution time can vary.

An attacker can collect many measurements and attempt to infer the bits of the private key.

Therefore:

> Cryptographic implementations should be designed to minimize secret-dependent timing differences.

---

# 2.5 Cache and Branch-Prediction Attacks

Modern processors use caches and branch prediction to improve performance.

These optimizations can accidentally reveal information about what a cryptographic program is doing.

For example:

```text
Secret Data
    ↓
Memory Access
    ↓
CPU Cache
    ↓
Different Timing
    ↓
Attacker Observation
```

An attacker may use carefully designed measurements to infer secret-dependent behavior.

This is particularly important in environments where an attacker can run code on the same machine or processor.

---

# 2.6 Power Analysis Attacks

Cryptographic operations performed by smart cards and embedded devices consume electrical power.

The power consumption can vary depending on the operations being performed.

```text
Smart Card
    │
    ▼
Cryptographic Operation
    │
    ▼
Power Consumption Pattern
    │
    ▼
Attacker Measurements
```

By collecting many power traces, an attacker may infer information about the secret key.

These attacks are commonly known as:

* Simple Power Analysis (SPA)
* Differential Power Analysis (DPA)

---

# 2.7 Fault Attacks

An attacker may deliberately cause a device to perform an incorrect computation.

For example:

```text
Normal computation
       ↓
Correct result

Induced fault
       ↓
Incorrect result
```

Comparing correct and faulty results can reveal information about secret cryptographic values.

This is known as a **fault-injection attack**.

---

# 2.8 Keys in Memory

Cryptographic keys must eventually exist in computer memory while they are being used.

For example:

```text
Encrypted File
      │
      ▼
Program
      │
      ▼
Secret Key in RAM
      │
      ▼
Decryption
```

If an attacker obtains a memory dump while the key is present, they may search for the key.

Cryptographic keys often have high entropy and appear random, which can make them distinguishable from normal structured data when combined with knowledge of the application and key format.

Therefore:

> **Protecting keys in memory is an important part of cryptographic security.**

Systems should minimize how long sensitive keys remain in memory and use appropriate operating-system and hardware protections.

---

# 2.9 Van Eck Radiation

Electronic devices can unintentionally emit electromagnetic radiation.

In some circumstances, an attacker can use specialized equipment to capture these emissions and reconstruct information being processed or displayed.

This idea became widely known through research often associated with **Van Eck phreaking**.

Historically, researchers demonstrated that CRT monitor emissions could potentially reveal displayed information.

Conceptually:

```text
CRT Monitor
    │
    │ Electromagnetic Emissions
    ▼
Attacker Receiver
    │
    ▼
Reconstructed Display
```

This demonstrates an important security principle:

> Information can leak through physical side channels even when the network communication itself is encrypted.

---

# 3. Network Authentication Protocols

Authentication becomes much more difficult when communication happens over an insecure network.

---

# 3.1 Standalone Authentication

Suppose a user logs into a local computer.

```text
User
  │
  ▼
Local Computer
  │
  ▼
Password Verification
```

The system can directly compare the entered password with the stored password-derived value.

The attacker does not necessarily control the communication channel between the keyboard and authentication system.

---

# 3.2 Network Authentication

Now imagine Alice authenticating to Bob across the Internet.

```text
Alice ───────── Internet ─────────► Bob
                    ▲
                    │
                  Trudy
```

Trudy may be able to:

* Read messages
* Delete messages
* Modify messages
* Delay messages
* Replay messages
* Inject new messages

Therefore, a network authentication protocol must assume that the communication channel is hostile.

---

# 3.3 Simple Authentication

A very basic authentication protocol might look like:

```text
Alice ───── "I am Alice" ─────► Bob
```

Bob obviously cannot trust this.

Anyone can claim to be Alice.

A better approach might send some authentication information:

```text
Alice ───── Authentication Data ─────► Bob
```

But if the authentication data remains the same every time, an attacker can record it.

---

# 3.4 Replay Attack

Suppose Alice sends:

```text
Alice → Authentication Message
```

Trudy records it.

Later:

```text
Trudy → Same Authentication Message → Bob
```

If Bob accepts it, Trudy has successfully impersonated Alice.

Therefore:

> **Authentication messages must include freshness.**

---

# 3.5 Challenge-Response Authentication

A common solution is **challenge-response authentication**.

Bob generates a random challenge called a **nonce**.

Nonce means:

> **Number used once**

The protocol can look like:

```text
Bob                         Alice

 │──── Random Nonce ───────►│
 │                          │
 │                    Uses Secret Key
 │                          │
 │◄──── Response ───────────│
 │                          │
 ▼
Verify Response
```

Because the nonce is fresh and unpredictable, an old response should not work against a new challenge.

---

# 3.6 Why Nonces Defeat Replay

Suppose Trudy records:

```text
Challenge A
Response A
```

Later Bob generates:

```text
Challenge B
```

Trudy only knows the correct response to **Challenge A**.

```text
Old Response A
       ↓
Challenge B
       ↓
Does not match
```

Therefore, the replay attack fails.

This is why random nonces are fundamental to many authentication protocols.

---

# 3.7 One-Way Authentication

Suppose Bob wants to authenticate Alice.

Bob sends a challenge:

[
R
]

Alice calculates a response using their shared secret key (K):

[
Response = MAC_K(R)
]

Bob calculates the same value.

```text
Bob                            Alice

Nonce R ──────────────────────►

        MAC_K(R)
◄──────────────────────────────

Verify MAC
```

If the response is correct, Bob gains confidence that Alice possesses the secret key.

This is **one-way authentication** because Bob authenticates Alice.

---

# 3.8 Mutual Authentication

Sometimes both parties need to authenticate each other.

```text
Alice ←──── Authentication ────► Bob
```

This is called **mutual authentication**.

A naive design might simply use similar challenge-response operations in both directions.

However, careless protocol design can create a **reflection attack**.

---

# 3.9 Reflection Attack

Suppose Trudy wants to impersonate Alice to Bob.

She opens a connection to Bob and receives a challenge.

Instead of solving the challenge, Trudy opens another connection to Bob and uses Bob's challenge as a challenge in the second connection.

```text
Connection 1:
Bob ── Challenge A ──► Trudy

Connection 2:
Trudy ── Challenge A ──► Bob
```

Bob may respond to his own challenge.

Trudy can potentially reflect that response back into the first connection.

```text
Bob ── Response A ──► Trudy
                       │
                       └──► Bob (Connection 1)
```

This demonstrates why authentication protocols must be carefully designed.

---

# 3.10 Secure Symmetric Mutual Authentication

One way to prevent reflection problems is to include the **identities and roles** of the communicating parties inside the authenticated data.

For example:

```text
Bob → Alice:
"Bob challenges Alice with nonce X"

Alice → Bob:
"Alice responds to Bob's challenge X"
```

The identity information binds the response to a particular protocol role.

Conceptually:

[
MAC_K("Alice", "Bob", nonce)
]

Now an attacker cannot simply reflect the same response in another context without changing the authenticated data.

---

# 3.11 Public-Key Authentication

Public-key cryptography can also be used for authentication.

Alice can sign a challenge using her private key:

```text
Bob
 │
 │ Challenge
 ▼
Alice
 │
 │ Signature(Challenge)
 ▼
Bob
 │
 ▼
Verify using Alice's Public Key
```

This allows Bob to verify that the challenge was signed using the private key corresponding to Alice's public key.

---

# 3.12 Problems With Naive Public-Key Authentication

A poorly designed protocol may allow an attacker to trick a system into signing or decrypting arbitrary data.

For example:

```text
Attacker
   │
   │ Arbitrary Challenge
   ▼
Alice
   │
   │ Signs / Decrypts
   ▼
Attacker
```

The resulting signature or decrypted information may help the attacker attack the cryptosystem.

Therefore, encryption and signing operations should not be treated as interchangeable.

---

# 3.13 Separate Keys for Different Purposes

A safer design uses separate key pairs for different cryptographic purposes.

For example:

```text
Alice

Encryption Key Pair
├── Public Encryption Key
└── Private Decryption Key

Signature Key Pair
├── Public Verification Key
└── Private Signing Key
```

The separation provides **key separation**.

The principle is:

> **Do not use the same cryptographic key for unrelated purposes when the protocol can use separate keys.**

This reduces the risk that an attacker can turn one type of cryptographic operation into another type of attack.

---

# 4. Session Keys and Key Agreement

Long-term keys are useful for establishing trust, but using the same key for every communication session creates risks.

A **session key** is a temporary symmetric key generated for a particular communication session.

---

# 4.1 Purpose of Session Keys

Suppose Alice and Bob communicate using one permanent key:

```text
Alice ───── Permanent Key ───── Bob
```

If that key is eventually compromised, many communications may be affected.

Instead, they can create a new key for each session:

```text
Session 1 → Key A
Session 2 → Key B
Session 3 → Key C
```

```text
Long-Term Key
      │
      ▼
Authentication / Key Agreement
      │
      ├── Session Key A
      ├── Session Key B
      └── Session Key C
```

Session keys limit the **blast radius** of a compromise.

---

# 4.2 Perfect Forward Secrecy

**Perfect Forward Secrecy (PFS)** provides protection for previously established sessions even if a long-term private key is compromised later.

Consider:

```text
2026
Session A → Encrypted Data
Session B → Encrypted Data
Session C → Encrypted Data

2027
Long-Term Key Stolen
```

Without forward secrecy, an attacker who recorded the old traffic might be able to decrypt it.

With PFS:

```text
Long-Term Key Stolen
        ↓
Future authentication affected
        ↓
Past session keys remain secret
```

This is a major security advantage.

---

# 4.3 Naive Session-Key Protocol

Consider a simple design.

Alice generates a random session key:

[
K_s
]

She encrypts it using a long-term key:

[
C = Encrypt(K_s, K_{master})
]

She sends it to Bob.

```text
Alice
 │
 │ Encrypted Session Key
 ▼
Bob
```

This works initially.

But suppose an attacker records the communication:

```text
Encrypted Session Key 1
Encrypted Session Key 2
Encrypted Session Key 3
...
```

Years later, the master key is stolen.

The attacker can use the master key to recover previously transmitted session keys.

```text
Master Key Compromised
        ↓
Old Encrypted Session Keys
        ↓
Recover Old Session Keys
        ↓
Decrypt Recorded Traffic
```

Therefore, this approach does **not** provide perfect forward secrecy.

---

# 4.4 Ephemeral Diffie–Hellman

**Ephemeral Diffie–Hellman** provides a better approach.

The word **ephemeral** means temporary or short-lived.

Alice generates a temporary private value:

[
a
]

Bob generates:

[
b
]

They exchange the corresponding public values.

```text
Alice                         Bob

Temporary a                  Temporary b

A = g^a mod p  ────────────►
               ◄──────────── B = g^b mod p
```

Both calculate:

[
K = g^{ab} \mod p
]

This produces the session key.

---

# 4.5 Why Ephemeral Keys Provide Forward Secrecy

After the session ends, Alice and Bob securely discard the temporary private values:

```text
Alice's a → Deleted
Bob's b   → Deleted
```

The session key was derived from:

[
g^{ab}
]

but the temporary secrets **a** and **b** are no longer available.

Therefore, even if a long-term authentication key is stolen later, the attacker should not be able to reconstruct the old Diffie–Hellman session secrets from previously recorded public values, assuming the underlying cryptography and implementation remain secure.

---

# 4.6 Preventing Man-in-the-Middle Attacks

Plain Diffie–Hellman alone is vulnerable to MitM attacks.

Therefore, ephemeral Diffie–Hellman must be **authenticated**.

A common design is:

```text
Ephemeral Diffie–Hellman
          +
Authentication
          ↓
Authenticated Key Agreement
```

For example, a party can digitally sign the ephemeral Diffie–Hellman parameters.

This gives both:

* **Authentication**
* **Perfect Forward Secrecy**

---

# 4.7 Complete Session Establishment

A simplified modern protocol can be viewed as:

```text
1. Authenticate identities
          ↓
2. Exchange ephemeral public values
          ↓
3. Calculate shared secret
          ↓
4. Derive session keys
          ↓
5. Encrypt application data
          ↓
6. Delete temporary secrets
```

The result is:

```text
Long-Term Identity Keys
          │
          ▼
    Authentication
          │
          ▼
Ephemeral DH Exchange
          │
          ▼
    Shared Secret
          │
          ▼
    Session Key(s)
          │
          ▼
   Secure Data Transfer
```



