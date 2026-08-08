# Kerberos Authentication in Active Directory

**Kerberos** is the primary authentication protocol used by Active Directory.

It allows users and computers to authenticate to network services **without repeatedly sending their passwords across the network**.

Instead of sending a password every time a user accesses a service, Kerberos uses **tickets** and **symmetric cryptography** to prove identity.

A simplified view is:

```text
User
 │
 │ Authenticate
 ↓
KDC
 │
 │ TGT
 ↓
User
 │
 │ Request service ticket
 ↓
KDC
 │
 │ Service Ticket
 ↓
User
 │
 │ Present ticket
 ↓
Network Service
```

Examples of services that can use Kerberos include:

* SMB
* LDAP
* HTTP
* SQL Server
* Other Kerberos-enabled network services

---

# 1. Core Cryptographic Concepts

## 1.1 What Problem Does Kerberos Solve?

Imagine two computers communicating over an untrusted network.

An attacker might be able to:

* Listen to network traffic
* Capture packets
* Modify packets
* Replay previously captured messages

Kerberos provides a way for a client and a network service to establish trust without sending the user's password across the network for every request.

It uses a trusted third party called the **Key Distribution Center (KDC)**.

The basic idea is:

```text
Client                    KDC                    Service
  │                        │                        │
  │──── Authentication ───→│                        │
  │                        │                        │
  │←────── TGT ────────────│                        │
  │                        │                        │
  │──── Service Request ──→│                        │
  │                        │                        │
  │←── Service Ticket ─────│                        │
  │                        │                        │
  │──────────────── Service Ticket ────────────────→│
  │                        │                        │
  │←────────────── Authenticated Session ──────────│
```

The KDC acts as the trusted authority that helps the client and service authenticate each other.

---

# 1.2 Symmetric-Key Cryptography

Kerberos primarily uses **symmetric-key cryptography**.

With symmetric cryptography, the same secret key is used to protect and verify data.

Conceptually:

```text
Plaintext
    │
    │ Secret Key
    ↓
 Encryption
    │
    ↓
Ciphertext
```

The receiver needs the appropriate secret key to decrypt or verify the protected information.

In Active Directory, long-term secret keys are associated with accounts such as:

* User accounts
* Computer accounts
* Service accounts
* The special `krbtgt` account

Kerberos then uses temporary **session keys** for individual authentication sessions.

---

# 1.3 Kerberos and the Needham-Schroeder Model

Kerberos is historically based on ideas from the **Needham-Schroeder symmetric-key authentication protocol**.

The central idea is:

> A trusted third party can help two parties establish a secure session without requiring them to already share a direct secret key.

Kerberos extends this concept using tickets, authenticators, timestamps, and session keys.

---

# 1.4 Kerberos vs HTTPS/TLS

Kerberos and HTTPS both provide authentication and secure communication, but they work differently.

### HTTPS

HTTPS uses **TLS (Transport Layer Security)**.

TLS commonly uses:

* Public-key cryptography
* Symmetric cryptography
* X.509 certificates
* Certificate Authorities (CAs)

For example:

```text
Browser
   │
   │ HTTPS
   ↓
Web Server
   │
   └── X.509 Certificate
           │
           ↓
      Certificate Authority
```

The certificate helps the browser verify the identity of the server.

---

### Kerberos

Kerberos does not rely on the public web PKI model for its normal authentication process.

Instead, it relies on a trusted **KDC**.

```text
Kerberos

Client
  │
  ↓
KDC
  │
  ↓
Tickets
  │
  ↓
Service
```

The important difference is:

| HTTPS/TLS                                                | Kerberos                                    |
| -------------------------------------------------------- | ------------------------------------------- |
| Uses TLS                                                 | Uses Kerberos protocol                      |
| Uses public-key cryptography during authentication setup | Primarily uses symmetric cryptography       |
| Uses X.509 certificates                                  | Uses tickets                                |
| Trust commonly comes from Certificate Authorities        | Trust comes from the KDC                    |
| Commonly used for web communication                      | Commonly used for enterprise authentication |

> Kerberos and TLS are not competitors. They can be used together. For example, an application may use Kerberos for user authentication while TLS protects the network connection.

---

# 2. Core Kerberos Components

Several components are essential to understanding Kerberos.

---

# 2.1 Key Distribution Center (KDC)

The **Key Distribution Center (KDC)** is the central trusted authority in a Kerberos realm.

In Active Directory, the KDC runs as a service on Domain Controllers.

The KDC is responsible for issuing Kerberos tickets.

It logically contains two main services:

```text
                 KDC
                  │
          ┌───────┴────────┐
          ↓                ↓
 Authentication      Ticket Granting
    Server (AS)        Server (TGS)
          │                │
          ↓                ↓
        TGT             Service Ticket
```

---

## Authentication Server (AS)

The **Authentication Server (AS)** handles the initial authentication.

Its main job is to verify the user's identity and provide a **Ticket Granting Ticket (TGT)**.

```text
Client
  │
  │ AS-REQ
  ↓
Authentication Server
  │
  │ AS-REP
  ↓
TGT
```

---

## Ticket Granting Server (TGS)

The **Ticket Granting Server (TGS)** issues tickets for individual services.

The client presents its TGT to the TGS and requests access to a particular service.

```text
Client
  │
  │ TGT + Service Request
  ↓
TGS
  │
  │ Service Ticket
  ↓
Client
```

---

# 2.2 The `krbtgt` Account

Active Directory contains a special account named:

```text
krbtgt
```

The account is associated with the secret keys used by the KDC to protect Kerberos ticket information.

For example, the TGT is protected using a key associated with the KDC/`krbtgt` account.

This is why protecting the `krbtgt` account and its associated secrets is extremely important.

> **Security Note:** Compromise of the `krbtgt` secret can have serious consequences for the security of the entire Active Directory domain.

---

# 2.3 Realm

A **Kerberos realm** is a logical authentication boundary containing principals that are managed by a particular KDC.

In traditional Kerberos environments, realms are commonly written in uppercase.

For example:

```text
EXAMPLE.LAB
```

In Active Directory, the Kerberos realm normally corresponds to the DNS domain name, represented in uppercase:

```text
example.lab
      ↓
EXAMPLE.LAB
```

So:

```text
Active Directory Domain:
example.lab

Kerberos Realm:
EXAMPLE.LAB
```

---

# 2.4 Principals

A **principal** is an identity recognized by Kerberos.

Principals can represent:

* Users
* Computers
* Services

A traditional Kerberos principal can be represented as:

```text
primary/instance@REALM
```

For example:

```text
HTTP/web01.example.lab@EXAMPLE.LAB
```

---

## User Principals

A user principal represents an individual user.

For example:

```text
leo@example.lab
```

In Active Directory, this is commonly represented as the user's **User Principal Name (UPN)**.

The UPN format is:

```text
username@domain
```

Example:

```text
leo@example.lab
```

---

## Service Principals

A service principal represents a network service that supports Kerberos authentication.

Examples include:

```text
HTTP/web01.example.lab
LDAP/dc01.example.lab
MSSQLSvc/sql01.example.lab
```

These identities are represented in Active Directory using **Service Principal Names (SPNs)**.

---

## What Is an SPN?

An **SPN (Service Principal Name)** is an identifier that associates a service with an account in Active Directory.

For example:

```text
HTTP/web01.example.lab
```

can identify an HTTP service running on a particular host.

SPNs are important because the client uses them when requesting a Kerberos service ticket.

A simplified process is:

```text
Client
  │
  │ "I need a ticket for HTTP/web01"
  ↓
KDC
  │
  ↓
Service Ticket
```

---

## Host Principals

Computer accounts can also participate in Kerberos.

For example:

```text
DC01$
WEB01$
CLIENT01$
```

These computer accounts have associated secrets used for authentication.

---

# 3. Kerberos Tickets

Tickets are one of the most important concepts in Kerberos.

There are two major tickets you should understand:

1. **Ticket Granting Ticket (TGT)**
2. **Service Ticket (ST)**

---

# 3.1 Ticket Granting Ticket (TGT)

The **TGT** is obtained after successful initial authentication with the Authentication Server.

It essentially says:

> "The KDC has authenticated this client."

The TGT is protected using a key known to the KDC.

The client does **not** normally decrypt and read the TGT itself.

Conceptually:

```text
Client
  │
  │ Initial authentication
  ↓
KDC
  │
  │ TGT
  ↓
Client
```

The TGT can later be presented to the TGS to request service tickets.

---

# 3.2 Service Ticket

A **Service Ticket (ST)** is issued for a particular network service.

For example, suppose a user wants to access an SMB service:

```text
SMB on DC01
```

The client asks the TGS for a ticket associated with the appropriate service identity.

The resulting service ticket is protected using a key associated with the target service account.

The client then presents the service ticket to the service.

```text
Client
  │
  │ Service Ticket
  ↓
SMB Service
```

---

# 3.3 Why Are There Two Types of Tickets?

You may wonder:

> Why doesn't Kerberos simply issue one ticket?

The separation makes Kerberos more efficient.

After the user initially authenticates, the TGT can be reused to request tickets for different services.

For example:

```text
                 TGT
                  │
        ┌─────────┼─────────┐
        ↓         ↓         ↓
       SMB       LDAP      HTTP
        │         │         │
        ↓         ↓         ↓
       ST        ST        ST
```

The user does not need to enter their password every time they access another service.

---

# 4. Kerberos Authentication Flow

The Kerberos process can be divided into three major phases:

```text
Phase 1
AS Exchange
     ↓
Obtain TGT

Phase 2
TGS Exchange
     ↓
Obtain Service Ticket

Phase 3
AP Exchange
     ↓
Authenticate to Service
```

Let's examine each phase.

---

# 5. Phase 1 — AS Exchange

The first phase is the **Authentication Service (AS) exchange**.

The objective is:

> Authenticate the user and obtain a TGT.

---

## Step 1: AS-REQ

The client sends an **AS-REQ (Authentication Service Request)** to the KDC.

Conceptually:

```text
Client
   │
   │ AS-REQ
   ↓
Authentication Server
```

The request contains information such as:

* Client identity
* Kerberos realm
* Requested service (`krbtgt`)
* Supported encryption types
* Nonce
* Ticket options

---

## Pre-Authentication

Modern Active Directory normally requires **Kerberos pre-authentication**.

The client proves knowledge of its long-term secret by providing encrypted pre-authentication data, commonly including a timestamp.

The general idea is:

```text
User's Secret Key
       +
Current Timestamp
       ↓
Encrypted Pre-Authentication Data
       ↓
KDC
```

The KDC can verify this information using the user's stored secret.

This helps prevent attackers from requesting authentication responses for arbitrary users and then easily performing offline password guessing.

---

## Clock Synchronization

Kerberos relies heavily on timestamps.

Therefore, the client and KDC must have reasonably synchronized clocks.

If the clocks differ too much, authentication can fail with a **clock skew** error.

This is why Windows domain environments typically use time synchronization mechanisms.

---

## Wrong Password

If the user provides an incorrect password, the client cannot correctly produce the required pre-authentication data.

The KDC therefore rejects the authentication attempt.

---

# 6. AS-REP

If authentication succeeds, the KDC sends an **AS-REP (Authentication Service Reply)**.

Conceptually:

```text
Authentication Server
        │
        │ AS-REP
        ↓
      Client
```

The response contains two important pieces of information:

### TGT

The TGT is protected with the KDC's secret key.

The client stores it but cannot normally decrypt its protected contents.

### Client/KDC Session Key

The AS-REP also contains information encrypted using a key derived from the user's long-term secret.

The client can decrypt this portion and obtain a temporary session key used during communication with the TGS.

---

# 7. AS-REP Roasting

There is an important security concept associated with Kerberos pre-authentication.

If an account is configured so that **Kerberos pre-authentication is not required**, an attacker may be able to request an AS-REP response for that account without first proving knowledge of its password.

The encrypted response can potentially be subjected to **offline password guessing**.

This technique is known as:

> **AS-REP Roasting**

The important distinction is:

```text
Normal Kerberos
      ↓
Pre-authentication required

AS-REP Roasting condition
      ↓
Pre-authentication disabled
      ↓
Obtain AS-REP
      ↓
Offline password cracking
```

This is why disabling Kerberos pre-authentication unnecessarily is dangerous.

---

# 8. Phase 2 — TGS Exchange

The second phase begins when the client wants to access a particular service.

For example:

```text
User
 ↓
Needs LDAP access
 ↓
Requests LDAP service ticket
```

---

## Step 2: TGS-REQ

The client sends a **TGS-REQ (Ticket Granting Service Request)** to the TGS.

The request contains:

* The TGT
* The requested service's SPN
* An authenticator
* Ticket options

For example:

```text
ldap/dc01.example.lab
```

The authenticator contains client information and a timestamp protected using the appropriate session key.

Conceptually:

```text
Client
  │
  │ TGT + Authenticator
  │ + Service SPN
  ↓
TGS
```

---

# 9. TGS-REQ Processing

The TGS verifies the request.

Conceptually:

```text
TGS
 │
 ├── Validate TGT
 │
 ├── Recover session information
 │
 ├── Validate Authenticator
 │
 └── Identify requested service
```

If everything is valid, the TGS creates a Service Ticket.

---

# 10. TGS-REP

The TGS responds with a **TGS-REP (Ticket Granting Service Reply)**.

The response contains:

* A Service Ticket
* Information encrypted for the client
* A Service Session Key

Conceptually:

```text
TGS
 │
 │ TGS-REP
 ↓
Client
 │
 ├── Service Ticket
 └── Service Session Key
```

The Service Ticket is intended for the target service.

---

# 11. Kerberoasting

Service accounts that have SPNs registered in Active Directory can be associated with Kerberos service tickets.

If a service account uses a weak password, an attacker with appropriate access may be able to request a service ticket and perform offline password guessing against protected ticket material.

This technique is known as:

> **Kerberoasting**

The basic concept is:

```text
Service Account
      │
      ↓
Registered SPN
      │
      ↓
Service Ticket
      │
      ↓
Offline password guessing
```

The important security lesson is:

> **Service accounts should use strong, unique credentials and should have only the privileges they require.**

---

# 12. Phase 3 — AP Exchange

The final phase is the **Application (AP) exchange**.

The client now has a Service Ticket and can present it directly to the target service.

For example:

```text
Client
   │
   │ Service Ticket
   ↓
LDAP Server
```

---

# 13. AP-REQ

The client sends an **AP-REQ (Application Request)** to the target service.

It contains:

* Service Ticket
* Authenticator

The service can decrypt the Service Ticket using its own secret key.

It then obtains the Service Session Key and uses that key to verify the authenticator.

Conceptually:

```text
Client
  │
  │ AP-REQ
  │
  │ Service Ticket
  │ +
  │ Authenticator
  ↓
Service
```

---

# 14. Service Authentication

The target service performs several checks.

It verifies that:

1. The Service Ticket was issued for the service.
2. The ticket is valid.
3. The authenticator is valid.
4. The timestamp is acceptable.
5. The client identity matches the ticket information.

If everything is correct, the service accepts the authentication.

---

# 15. AP-REP and Mutual Authentication

Kerberos can also provide **mutual authentication**.

Without mutual authentication:

```text
Client → Service
"Here is my valid ticket."
```

The service verifies the client.

With mutual authentication:

```text
Client ↔ Service
```

Both sides can verify the identity of the other.

The service responds with an **AP-REP (Application Reply)** containing information protected using the Service Session Key.

The client verifies the response and can therefore confirm that it is communicating with the expected service.

---

# 16. Complete Kerberos Flow

Putting everything together:

```text
                    KDC
             ┌───────┴───────┐
             │               │
             AS              TGS
             │               │
             └───────┬───────┘
                     │
                     │
Client ──────────────┤
  │                  │
  │ 1. AS-REQ        │
  ├─────────────────→│
  │                  │
  │ 2. AS-REP + TGT  │
  │←─────────────────┤
  │                  │
  │ 3. TGS-REQ       │
  ├─────────────────→│
  │                  │
  │ 4. TGS-REP + ST  │
  │←─────────────────┤
  │
  │ 5. AP-REQ
  ├────────────────────────→ Service
  │
  │ 6. AP-REP (optional)
  │←────────────────────────
  │
  ↓
Authenticated Session
```

---

# 17. Why Kerberos Does Not Send the Password Every Time

One of Kerberos' biggest advantages is that the user's password is not repeatedly transmitted across the network.

Instead:

```text
Initial Authentication
        ↓
       TGT
        ↓
Service Ticket
        ↓
Access Service
```

For example, a user may access several services:

```text
              TGT
               │
       ┌───────┼────────┐
       ↓       ↓        ↓
      SMB     LDAP     HTTP
       │       │        │
      ST      ST        ST
```

The user does not need to repeatedly send their password to each service.

---

# 18. Important Kerberos Terms

| Term              | Meaning                                                       |
| ----------------- | ------------------------------------------------------------- |
| **KDC**           | Central Kerberos authority                                    |
| **AS**            | Authentication Server                                         |
| **TGS**           | Ticket Granting Server                                        |
| **TGT**           | Ticket used to request service tickets                        |
| **ST**            | Ticket used to authenticate to a specific service             |
| **Principal**     | Identity known to Kerberos                                    |
| **Realm**         | Kerberos authentication boundary                              |
| **SPN**           | Identifier for a Kerberos-enabled service                     |
| **UPN**           | User Principal Name                                           |
| **Authenticator** | Data used to prove possession of a session key                |
| **Session Key**   | Temporary key used for a particular authentication session    |
| **`krbtgt`**      | Special AD account associated with KDC ticket-protection keys |
| **AS-REQ**        | Request sent to the Authentication Server                     |
| **AS-REP**        | Authentication Server response                                |
| **TGS-REQ**       | Request for a Service Ticket                                  |
| **TGS-REP**       | TGS response containing a Service Ticket                      |
| **AP-REQ**        | Request sent to the target service                            |
| **AP-REP**        | Optional response used for mutual authentication              |

---

# 19. Useful Kerberos Commands

Windows provides several commands that are useful when investigating Kerberos.

---

## `whoami /upn`

The command:

```cmd
whoami /upn
```

displays the current user's **User Principal Name (UPN)**.

For example:

```text
leo@example.lab
```

This can help identify the user's domain.

---

# 20. `klist`

The `klist` command displays Kerberos tickets associated with the current logon session.

```cmd
klist
```

You may see information about:

* TGTs
* Service Tickets
* Ticket expiration times
* Kerberos service principals

Conceptually:

```text
Cached Kerberos Tickets
        │
        ├── krbtgt/EXAMPLE.LAB
        ├── cifs/dc01.example.lab
        └── ldap/dc01.example.lab
```

This is particularly useful when troubleshooting Kerberos authentication.

---

# 21. `setspn`

The `setspn` command can be used to view and manage Service Principal Names.

For example:

```cmd
setspn -L dc01
```

lists the SPNs associated with the specified computer account.

You might see entries such as:

```text
HOST/dc01.example.lab
LDAP/dc01.example.lab
RestrictedKrbHost/dc01.example.lab
```

This helps administrators understand which services are registered for Kerberos authentication.

---

# 22. Common Kerberos Security Attacks

Several important Active Directory attacks are directly related to Kerberos.

### AS-REP Roasting

Occurs when an account does not require Kerberos pre-authentication.

```text
No pre-auth
    ↓
AS-REP
    ↓
Offline password guessing
```

### Kerberoasting

Targets service accounts associated with SPNs.

```text
SPN
 ↓
Service Ticket
 ↓
Offline password guessing
```

### Golden Ticket

Involves forging Kerberos TGTs using compromised `krbtgt` secrets.

```text
Compromised krbtgt secret
        ↓
Forged TGT
        ↓
Potential domain-wide impact
```

### Silver Ticket

Involves forging a service ticket using a compromised service account or computer account secret.

```text
Compromised service secret
        ↓
Forged Service Ticket
        ↓
Access to targeted service
```

These attacks demonstrate why protecting **KDC secrets, service accounts, and privileged credentials** is critical.

---

# 23. A Simple Mental Model

If the Kerberos protocol seems complicated, remember this simple sequence:

```text
1. Authenticate
       ↓
2. Get TGT
       ↓
3. Ask TGS for a service ticket
       ↓
4. Receive Service Ticket
       ↓
5. Present Service Ticket to service
       ↓
6. Service verifies the ticket
       ↓
7. Authenticated session
```

Or, even shorter:

```text
Password
   ↓
Authentication
   ↓
TGT
   ↓
Service Ticket
   ↓
Service Access
```

---

# 24. Final Takeaways

The most important concepts to remember are:

* **Kerberos is the primary authentication protocol used by Active Directory.**
* It uses **tickets** instead of repeatedly sending passwords to services.
* The **KDC** is the trusted authority responsible for issuing tickets.
* The KDC logically consists of the **Authentication Server (AS)** and **Ticket Granting Server (TGS)**.
* A **TGT** is used to request Service Tickets.
* A **Service Ticket** is used to authenticate to a specific service.
* **SPNs** identify Kerberos-enabled services.
* **Session keys** provide temporary cryptographic protection for authentication exchanges.
* **Timestamps** help protect against replay attacks and require reasonably synchronized clocks.
* **AS-REP Roasting** is associated with accounts that do not require Kerberos pre-authentication.
* **Kerberoasting** targets service accounts associated with SPNs.
* The `krbtgt` account is critical to the security of the Kerberos infrastructure in Active Directory.
* `klist` is useful for viewing cached Kerberos tickets.
* `setspn` is useful for investigating registered SPNs.
* Kerberos can provide **mutual authentication**, allowing both the client and service to verify each other.

> **The simplest way to remember Kerberos is: authenticate once, receive a TGT, exchange the TGT for Service Tickets, and use those tickets to access network services securely.**
