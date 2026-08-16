# Digital Signatures, Certificates, PKI, and PGP

Digital signatures and digital certificates are important parts of modern cryptography. They allow systems to verify **who created information**, **whether the information was modified**, and **whether a public key can be trusted**.

This chapter covers:

1. Fundamentals of Digital Signatures
2. Digital Signature Standards
3. Public-Key Trust and Digital Certificates
4. Trust Chains and Assurance Levels
5. Certificate Lifecycle Management
6. PGP and Decentralised Trust

---

# 1. Fundamentals of Digital Signatures

## 1.1 Symmetric vs. Asymmetric Authentication

There are two major approaches to cryptographic authentication.

### Symmetric Authentication

Symmetric authentication uses a **shared secret key**.

Both Alice and Bob know the same secret:

```text
Alice                         Bob
  │                            │
  │      Shared Secret Key     │
  └────────────────────────────┘
```

Alice can use the secret key to create a MAC, and Bob uses the same key to verify it.

The problem is that both parties possess the same secret.

Therefore, Bob cannot prove to a third party that **Alice specifically** created the message, because Bob also had the ability to create the same MAC.

---

### Asymmetric Authentication

Asymmetric cryptography uses a **private key and a public key**.

```text
Alice
 ├── Private Key → Keep Secret
 └── Public Key  → Share Publicly
```

Alice uses her **private key** to create a digital signature.

Other people use Alice's **public key** to verify it.

```text
Alice                         Bob

Private Key
    │
    ▼
Create Signature
    │
    │ Document + Signature
    └───────────────────────►
                              │
                              ▼
                       Alice's Public Key
                              │
                              ▼
                       Verify Signature
```

This provides an important advantage:

> Anyone can verify Alice's signature, but only someone possessing Alice's private key should be able to create a valid signature.

---

# 1.2 What Is a Digital Signature?

A **digital signature** is a cryptographic mechanism used to provide evidence about the origin and integrity of digital information.

It can provide:

* **Authentication** — evidence of who signed the data
* **Integrity** — evidence that the data was not modified
* **Nonrepudiation** — supports the ability to demonstrate that the signer approved the data

A digital signature is therefore similar in purpose to a handwritten signature, but it is based on mathematics rather than handwriting.

---

## Handwritten vs. Digital Signatures

A handwritten signature:

```text
Document
   +
Handwritten Signature
   ↓
Evidence of approval
```

A digital signature:

```text
Digital Document
      │
      ▼
Cryptographic Hash
      │
      ▼
Private-Key Operation
      │
      ▼
Digital Signature
```

The digital signature is mathematically connected to the document and the signer's private key.

---

# 1.3 What Can Digital Signatures Prove?

Suppose Alice signs a contract:

```text
Contract
   +
Alice's Digital Signature
```

Bob can use Alice's public key to verify the signature.

If verification succeeds, Bob gains confidence that:

1. The signature was generated using the corresponding private key.
2. The signed content has not been changed.
3. The signature is associated with Alice's public key.

Digital signatures are used in many applications, including:

* Electronic documents
* Software/code signing
* Email security
* Secure communications
* Financial transactions
* Authentication systems

---

# 1.4 Basic Signing Process

A simplified explanation is that Alice uses her private key to create a signature associated with the document.

Historically, some explanations describe this as:

```text
Document
   │
   ▼
Encrypt with Private Key
   │
   ▼
Digital Signature
```

The receiver can then use Alice's public key to verify it:

```text
Digital Signature
       │
       ▼
Alice's Public Key
       │
       ▼
Verification
```

However, real-world digital signature systems normally **do not encrypt the entire document with the private key**. Instead, they calculate a cryptographic hash of the document and sign that hash.

---

# 1.5 Why Hash the Document?

Imagine Alice wants to sign a 500 MB file.

Using a public-key algorithm directly on the entire file would be inefficient.

Instead, Alice calculates a short **message digest**:

```text
Large Document
      │
      ▼
Hash Function
      │
      ▼
Small Digest
```

She then signs the digest with her private key.

```text
Document
   │
   ▼
Hash
   │
   ▼
Digest
   │
   ▼
Private Key
   │
   ▼
Signature
```

The receiver performs the same hash operation on the received document.

```text
Received Document
       │
       ▼
     Hash
       │
       ▼
New Digest
```

The receiver then verifies the signature and checks that the digest matches.

---

# 1.6 Hash Functions

A cryptographic hash function converts data of arbitrary length into a fixed-size value.

For example:

```text
Large File
    │
    ▼
SHA Hash Function
    │
    ▼
Fixed-Length Digest
```

Important properties include:

* Small changes produce very different hash values.
* The original document cannot practically be reconstructed from the hash.
* It should be computationally difficult to find another message with the same hash.

Historical systems used algorithms such as **MD5** and **SHA-1**.

However, MD5 and SHA-1 are now considered cryptographically broken for many security applications.

Modern systems should use appropriate modern hash algorithms such as **SHA-256** or stronger algorithms where required.

---

# 1.7 Signature Verification

Suppose Alice signs a document.

### Alice

```text
Document
   │
   ▼
SHA-256
   │
   ▼
Digest
   │
   ▼
Alice's Private Key
   │
   ▼
Signature
```

Alice sends:

```text
Document + Signature
```

### Bob

Bob calculates the hash of the received document:

```text
Document
   │
   ▼
SHA-256
   │
   ▼
Digest A
```

He also verifies the signature using Alice's public key:

```text
Signature
   │
   ▼
Alice's Public Key
   │
   ▼
Digest B
```

Bob compares the results.

```text
Digest A = Digest B
       ↓
Signature Valid
```

If they do not match:

```text
Digest A ≠ Digest B
       ↓
Signature Invalid
```

This can indicate that the document was modified or that the signature is not valid.

---

# 1.8 Nonrepudiation and Stolen Private Keys

One of the difficult questions surrounding digital signatures is:

> What happens if Alice claims that her private key was stolen?

Suppose Alice signed a document using her private key.

Later she says:

> "I did not sign this. Someone stole my private key."

This creates both **technical and legal questions**.

A digital signature system cannot simply determine who physically controlled the private key at the time of signing.

This is why secure private-key management is extremely important.

Private keys should be:

* Protected from unauthorized access
* Stored securely
* Backed up appropriately
* Revoked when compromised
* Replaced when necessary

Digital signatures can provide strong cryptographic evidence, but legal nonrepudiation also depends on the surrounding policies, identity verification, key protection, and applicable laws.

---

# 2. Digital Signature Standards

Several algorithms have been developed for digital signatures.

Two important examples are:

* RSA
* DSA

Elliptic-curve systems also provide digital signature mechanisms such as **ECDSA**.

---

# 2.1 RSA Signatures

RSA was one of the first public-key algorithms suitable for digital signatures.

The basic concept is:

```text
Message
   │
   ▼
Hash
   │
   ▼
Private Key
   │
   ▼
Signature
```

Using simplified RSA notation, if the message hash is represented by (H(m)), the signature can be represented as:

[
s = H(m)^d \mod N
]

where:

* (H(m)) = message hash
* (d) = private exponent
* (N) = RSA modulus
* (s) = signature

---

## RSA Signature Verification

The receiver uses the public exponent:

[
H(m)' = s^e \mod N
]

The receiver also calculates the hash of the received message.

If the values match:

```text
Calculated Hash
      =
Recovered Signed Hash
      ↓
Signature Valid
```

Modern RSA signatures use standardized secure padding schemes such as **RSA-PSS**, rather than raw textbook RSA operations.

---

# 2.2 Digital Signature Algorithm (DSA)

**DSA**, or **Digital Signature Algorithm**, was standardized by NIST as a dedicated digital signature algorithm.

Unlike RSA, DSA was designed specifically for **digital signatures**.

It is not a general-purpose encryption algorithm.

Its security is based on the difficulty of the **discrete logarithm problem**.

---

## DSA Characteristics

DSA has several notable characteristics:

* Designed specifically for signatures
* Based on discrete logarithms
* Can benefit from precomputation in some environments
* Historically useful for constrained devices such as smart cards
* Signature verification can be slower than some RSA configurations

A major advantage of DSA was that it was made available without the same kind of licensing concerns associated with some earlier patented cryptographic technology.

---

# 2.3 ECDSA

**ECDSA**, or **Elliptic Curve Digital Signature Algorithm**, applies elliptic-curve mathematics to digital signatures.

It provides strong security with relatively small key sizes.

For example:

```text
Traditional public-key systems
        ↓
Large keys

Elliptic-curve systems
        ↓
Smaller keys for comparable security levels
```

ECDSA has been widely used in:

* TLS
* Digital certificates
* Secure applications
* Cryptographic libraries

---

# 3. Public-Key Trust and Digital Certificates

Public-key cryptography solves an important problem:

> How can Alice communicate securely with Bob without previously sharing a secret key?

But it introduces another problem:

> **How does Alice know that a particular public key actually belongs to Bob?**

Suppose Alice receives:

```text
Bob's Public Key
```

How does she know that it really belongs to Bob?

An attacker could send:

```text
Fake "Bob" Public Key
```

Alice might then unknowingly encrypt information for the attacker.

This is the **public-key trust problem**.

---

# 3.1 Certificate Authorities

A **Certificate Authority (CA)** helps solve this problem.

A CA acts as a **Trusted Third Party (TTP)** that binds a public key to an identity.

Conceptually:

```text
Bob
 │
 │ Public Key
 ▼
Certificate Authority
 │
 │ Verifies identity
 ▼
Digital Certificate
 │
 ▼
"Bob owns this public key"
```

The CA digitally signs the certificate.

Therefore, users can verify that the certificate was issued by a trusted CA.

---

# 3.2 What Is a Digital Certificate?

A digital certificate is essentially a digitally signed data structure that associates:

> **An identity + a public key**

A simplified certificate contains:

```text
Certificate
├── Subject
├── Subject Public Key
├── Issuer
├── Valid From
├── Valid Until
└── CA Digital Signature
```

---

# 3.3 Important Certificate Fields

### Issuer

The **Issuer** identifies the organization or CA that issued the certificate.

```text
Issuer → Certificate Authority
```

### Subject

The **Subject** identifies the entity to which the certificate belongs.

For example:

```text
Subject → example.com
```

or an organization/person depending on the certificate type.

### Time Limit

Certificates have a validity period.

For example:

```text
Valid From: January 1
Valid Until: January 1 next year
```

After the expiration date, the certificate should no longer be considered valid for its intended purpose.

---

# 3.4 Public Key Infrastructure (PKI)

**Public Key Infrastructure (PKI)** is the overall framework used to manage public-key certificates and trust.

PKI can include:

* Certificate Authorities
* Registration Authorities
* Digital certificates
* Certificate policies
* Key management
* Certificate revocation
* Trust stores

Conceptually:

```text
             PKI
              │
    ┌─────────┼─────────┐
    ▼         ▼         ▼
   CA         RA    Certificates
    │
    ▼
Trust Management
```

---

# 3.5 X.509 Certificates

**X.509** is a widely used standard for digital certificates.

X.509 certificates are commonly used for:

### HTTPS

Websites use certificates to authenticate domains and establish secure TLS connections.

```text
Browser ─────► https://example.com
                  │
                  ▼
             X.509 Certificate
```

### Email Security

Technologies such as **S/MIME** can use certificates for secure email.

### Code Signing

Software developers can digitally sign applications and updates.

The operating system or security software can then verify the signature and determine whether the software has been modified and whether the signer is associated with a trusted identity.

---

# 4. Trust Chains and Assurance Levels

A major question is:

> Why should we trust a Certificate Authority?

Operating systems and browsers contain lists of trusted **root CAs**.

A certificate can be linked back to one of these trusted roots.

This creates a **chain of trust**.

---

# 4.1 Low-Assurance Certificates

Low-assurance certificates typically require relatively limited identity verification.

For example, an automated process may verify control of an email address or domain.

The purpose is generally to provide basic assurance.

```text
Low Assurance
      ↓
Less identity verification
      ↓
Lower confidence in real-world identity
```

---

# 4.2 High-Assurance Certificates

High-assurance certificates require stronger identity verification.

Depending on the certificate type and policy, this may involve:

* Government-issued identification
* Organization verification
* In-person verification
* Multiple forms of evidence

The goal is to provide stronger assurance that the certificate actually belongs to the claimed entity.

```text
High Assurance
      ↓
More verification
      ↓
Higher confidence in identity
```

---

# 4.3 Certificate Classes

Historical certificate systems, including the VeriSign model referenced in the lecture, used different certificate classes to represent different assurance levels and intended uses.

A simplified progression is:

| Class   | General Purpose                            |
| ------- | ------------------------------------------ |
| Class 1 | Individual/email identity                  |
| Class 2 | Individual/organization information        |
| Class 3 | Stronger identity verification             |
| Class 4 | Business-to-business transactions          |
| Class 5 | Government or highly trusted organizations |

The exact certificate classes and commercial products have changed over time, so these historical categories should not be confused with today's certificate validation terminology.

---

# 4.4 Certificate Trust Chains

Certificates do not necessarily have to be signed directly by a root CA.

A hierarchy can be created.

For example:

```text
Root CA
   │
   ▼
Intermediate CA
   │
   ▼
Certificate
   │
   ▼
example.com
```

A more detailed model may include a **Registration Authority (RA)**.

```text
Root CA
   │
   ▼
Intermediate CA
   │
   ▼
Registration Authority
   │
   ▼
End-Entity Certificate
```

Each level provides evidence that the next certificate in the chain can be trusted.

---

# 4.5 Root Certificates

The top of the chain is normally a **root certificate**.

A root CA certificate is typically **self-signed**.

```text
Root Certificate
      │
      ├── signs Intermediate CA
      │
      ▼
Intermediate Certificate
      │
      ├── signs Website Certificate
      │
      ▼
Website Certificate
```

Operating systems and browsers maintain trusted root certificates.

If the certificate chain can be successfully validated back to a trusted root, the certificate can be considered trusted, subject to other validation requirements.

---

# 5. Certificate Lifecycle Management

Certificates are not permanent.

They have a lifecycle:

```text
Generate
   ↓
Issue
   ↓
Use
   ↓
Renew
   ↓
Expire
   ↓
Replace
```

A certificate may also be revoked before its normal expiration date.

---

# 5.1 Certificate Expiration

Certificates contain a defined validity period.

For example:

```text
Valid From: 2026-01-01
Valid Until: 2027-01-01
```

Why not make certificates valid forever?

Because security conditions change.

For example:

* Private keys can be stolen.
* Algorithms can become weaker.
* Certificate information can become outdated.
* Organizations can change ownership.
* Domains can change owners.
* Cryptographic standards can evolve.

Therefore, certificates must be periodically renewed.

Internet certificates historically had longer validity periods, but modern browser policies have progressively shortened maximum TLS certificate lifetimes.

---

# 5.2 Certificate Revocation

Sometimes a certificate must be invalidated **before its expiration date**.

For example:

```text
Certificate
    │
    ▼
Private Key Stolen
    │
    ▼
Certificate must be revoked
```

Another example is when a certificate was issued incorrectly.

The CA must communicate that the certificate should no longer be trusted.

---

# 5.3 Certificate Revocation Lists

A **Certificate Revocation List (CRL)** is a list of certificates that have been revoked.

Conceptually:

```text
CRL
├── Certificate A → Revoked
├── Certificate B → Revoked
├── Certificate C → Revoked
└── Certificate D → Revoked
```

A client can obtain the CRL and check whether a certificate appears on the list.

---

# 5.4 Polling Model

In a polling model, the client requests the latest revocation information.

```text
Client
  │
  │ Request CRL
  ▼
CA / CRL Distribution Point
  │
  │ CRL
  ▼
Client
```

The client periodically checks for updates.

---

# 5.5 Push Model

In a push model, revocation information is distributed to clients or systems when updates occur.

```text
CA
 │
 │ Revocation Update
 ▼
Clients
```

The lecture contrasts these approaches as:

* **Polling** → clients request revocation information.
* **Push** → the authority distributes revocation information.

In modern PKI, other mechanisms such as **OCSP** and browser-managed revocation systems may also be used.

---

# 5.6 Certificate Management Tools

Several tools can be used to work with certificates and keys.

## Java Keytool

Java provides the command-line tool:

```bash
keytool
```

It can be used to:

* Generate keys
* Create certificate requests
* Import certificates
* Export certificates
* Manage certificates and keys

Java uses a **keystore** to store cryptographic keys and certificates.

Conceptually:

```text
keytool
   │
   ▼
Keystore
   ├── Private Keys
   ├── Public Certificates
   └── Certificate Chains
```

---

## OpenSSL

**OpenSSL** is another widely used cryptographic toolkit.

It can be used to:

* Generate keys
* Generate certificate requests
* Inspect certificates
* Create test certificates
* Convert certificate formats
* Perform cryptographic operations
* Work with TLS

For example:

```bash
openssl x509 -in certificate.pem -text -noout
```

This can display information contained in an X.509 certificate.

---

# 6. Pretty Good Privacy (PGP)

**Pretty Good Privacy (PGP)** provides a different approach to public-key trust.

PGP was developed by **Phil Zimmermann** in 1991, initially becoming well known for secure email.

Unlike traditional PKI, PGP does not require all users to depend on a single centralized CA hierarchy.

Instead:

> **Users can make their own decisions about whom they trust.**

---

# 6.1 PGP Certificates

PGP certificates contain public-key information and can be signed by other users.

A simplified representation is:

```text
PGP Certificate
      │
      ├── User Identity
      ├── Public Key
      ├── User's Signature
      ├── Other User's Signature
      └── Another User's Signature
```

This allows multiple people to certify that they believe a particular public key belongs to a particular person.

This is fundamentally different from the typical X.509 model, where a certificate is issued within a CA hierarchy.

---

# 6.2 Self-Signed Certificates

PGP keys can be **self-signed**.

This means the key holder creates a signature over their own key information.

For example:

```text
Alice
  │
  ▼
Alice's Public Key
  │
  ▼
Alice signs her own key
```

However, a self-signature alone does not necessarily establish that Alice is a real-world identity.

Additional signatures from trusted people can provide more confidence.

---

# 6.3 Web of Trust

The **Web of Trust** is one of PGP's most important concepts.

Instead of:

```text
Everyone
    │
    ▼
Central CA
    │
    ▼
Trust
```

PGP can work more like:

```text
Alice ───── trusts ───── Bob
  │                       │
  │                       │
trusts                  trusts
  │                       │
  ▼                       ▼
Charlie                 David
```

People decide whom they trust.

If Alice personally knows Bob and trusts Bob's ability to verify identities, Alice may trust Bob's signature on another person's public key.

Bob becomes a **trusted introducer**.

---

# 6.4 Centralized vs Decentralized Trust

Traditional PKI can be viewed as a centralized or hierarchical trust model.

```text
            Root CA
              │
       ┌──────┴──────┐
       ▼             ▼
  Intermediate    Intermediate
       │             │
       ▼             ▼
   Certificate    Certificate
```

Users ultimately depend on a relatively small set of trusted root authorities.

PGP uses a more decentralized model:

```text
Alice ─── Bob
 │  \      /  │
 │   \    /   │
 │   Charlie  │
 │      │     │
 └──── David ─┘
```

Trust decisions can be distributed among users.

---

# 6.5 Advantages of the Web of Trust

The decentralized model can provide:

* Less dependence on a central CA
* Greater user control
* Multiple independent trust relationships
* Resistance to failure of a single authority
* Flexibility in deciding whom to trust

However, it also places more responsibility on users.

Users must understand:

* Whom they trust
* Why they trust them
* How keys were verified
* Whether a key has changed or been compromised

---

# 6.6 PGP vs X.509

| Feature                  | X.509 / PKI                   | PGP                                   |
| ------------------------ | ----------------------------- | ------------------------------------- |
| Trust Model              | Centralized/hierarchical      | Decentralized                         |
| Main Trust Authority     | CA                            | Users                                 |
| Certificates             | CA-issued                     | User-managed                          |
| Trust Relationships      | Certificate chains            | Web of Trust                          |
| Multiple User Signatures | Not the normal model          | Supported                             |
| Common Uses              | HTTPS, S/MIME, enterprise PKI | Secure email, personal key management |

Neither model is universally "better." They solve the trust problem using different approaches.



