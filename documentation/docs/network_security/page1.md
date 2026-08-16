# Access Control Models

Access control is one of the fundamental concepts in data and network security. It determines **who can access a resource, what they are allowed to do, and under what conditions**.

Different security models provide different ways of controlling access. Some models give control to the owner of a resource, while others enforce centrally defined security policies. The major models discussed here are **Discretionary Access Control (DAC), Mandatory Access Control (MAC), Bell-La Padula, BIBA, and Clark-Wilson**.

---

## 1. Discretionary Access Control (DAC)

**Discretionary Access Control (DAC)** is an access control model where the **owner of an object controls who can access it**.

### Ownership-Based Access

When a user creates an object, such as a file or folder, that user normally becomes its **owner**. The owner can then decide which other users or groups can:

* Read the object
* Write or modify the object
* Execute the object
* Delete or otherwise access the object

For example, if Alice creates a file called `report.txt`, Alice becomes the owner and can give Bob permission to read the file.

```text
Alice creates report.txt
        ↓
Alice becomes the owner
        ↓
Alice defines permissions
        ↓
Bob → Read
Charlie → Read + Write
David → No Access
```

This approach is called **discretionary** because the owner has discretion over how access is granted.

### Advantages of DAC

DAC is relatively **simple and flexible**. Users can share resources without requiring an administrator to change every permission.

For example, in a Linux system, a file owner can use commands such as:

```bash
chmod 640 report.txt
```

to control access to the file.

### Scalability Problems

Although DAC is easy to operate, managing permissions becomes increasingly difficult as a system grows.

Imagine an organization with:

* 50,000 users
* 10,000 computers
* Millions of files
* Thousands of applications

Managing individual permissions for every user and object can become extremely complicated.

Therefore, DAC is convenient for smaller environments but can become difficult to manage at very large scales.

### Security Concerns

DAC also relies heavily on the assumption that **users who create and manage objects can be trusted**.

A user with sufficient privileges may accidentally or intentionally grant access to sensitive information.

Administrators and superusers also have highly privileged permissions. If such an account is compromised, an attacker may gain extensive access to the system.

### Simple Example

Consider a shared folder:

```text
Owner: Alice

report.pdf
├── Alice    → Read + Write
├── Bob      → Read
├── Charlie  → No Access
└── Admin    → Full Access
```

The permissions are determined according to the owner's decisions and the system's permission mechanisms.

---

# 2. Mandatory Access Control (MAC)

**Mandatory Access Control (MAC)** is an access control model in which access decisions are determined by a **centrally defined security policy**, rather than being controlled by individual object owners.

This makes MAC more restrictive than DAC.

### External Administration

In MAC, a security administrator establishes the access policy.

The owner of a file generally **cannot simply change its security classification or grant access to anyone they choose**.

For example:

```text
Security Administrator
        ↓
Defines security policy
        ↓
Assigns classifications
        ↓
System enforces access rules
```

### Labels and Clearances

MAC commonly uses **security labels** for objects and **security clearances** for subjects.

An object could have a classification such as:

```text
Top Secret
Secret
Confidential
Unclassified
```

A user or process is assigned an appropriate clearance.

For example:

```text
User: Alice
Clearance: Secret

File A → Confidential
File B → Secret
File C → Top Secret
```

Alice may be permitted to access File A and File B but not File C, depending on the security policy.

The important idea is that **access is determined by security labels and centrally enforced rules**.

### Why MAC Is Useful

MAC is particularly useful in environments where confidentiality or integrity is extremely important.

Examples include:

* Military systems
* Government systems
* High-security networks
* Systems processing classified information

Unlike DAC, users cannot freely override the security policy simply because they own an object.

---

# 3. Bell-La Padula Model

The **Bell-La Padula (BLP) model** was developed in **1973** and is primarily concerned with **confidentiality**.

Its main goal is to prevent sensitive information from being disclosed to users who are not authorized to see it.

The model uses security classifications such as:

```text
Top Secret
    ↓
Secret
    ↓
Confidential
    ↓
Unclassified
```

The central idea is:

> **Protect secrets from being read or leaked to lower-security levels.**

## The Two Important Rules

### 3.1 No Read Up

The **Simple Security Property**, commonly remembered as **"No Read Up"**, prevents a subject from reading information at a higher classification level.

For example:

```text
Alice → Secret clearance

Confidential file → ✓ Can Read
Secret file       → ✓ Can Read
Top Secret file   → ✗ Cannot Read
```

A subject cannot read an object that is classified above the subject's clearance.

```text
Subject: Confidential
          ↓
      Cannot Read
          ↓
Object: Secret
```

This prevents lower-cleared users from accessing highly classified information.

### 3.2 No Write Down

The ***-property**, commonly remembered as **"No Write Down"**, prevents a subject from writing information to a lower security level.

For example:

```text
Alice → Secret clearance

Secret information
        ↓
    Cannot Write
        ↓
Confidential file
```

Why?

Suppose Alice can read a Secret document and then copies its contents into a Confidential document. The Secret information would effectively become available at a lower classification level.

Therefore, Bell-La Padula prevents this type of information leakage.

### Bell-La Padula Summary

| Rule              | Meaning                                               |
| ----------------- | ----------------------------------------------------- |
| **No Read Up**    | Cannot read data at a higher classification           |
| **No Write Down** | Cannot write sensitive data to a lower classification |

### Main Goal

**Bell-La Padula = Confidentiality**

It focuses on preventing **unauthorized disclosure of information**.

---

# 4. BIBA Model

The **BIBA model** takes a different approach from Bell-La Padula.

While Bell-La Padula focuses on **confidentiality**, BIBA focuses on **integrity**.

The goal is to prevent trusted or highly accurate information from being contaminated or modified by less trustworthy information.

### Integrity Levels

Consider the following integrity levels:

```text
High Integrity
      ↓
Medium Integrity
      ↓
Low Integrity
```

A high-integrity object is considered more trustworthy than a low-integrity object.

For example:

```text
Operating System Configuration
        ↓
   High Integrity

Downloaded File
        ↓
   Low Integrity
```

The BIBA model attempts to prevent low-integrity information from influencing high-integrity information.

## 4.1 No Read Down

The **Simple Integrity Axiom**, commonly remembered as **"No Read Down"**, prevents a subject from reading information at a lower integrity level.

For example:

```text
High-integrity subject
        ↓
Cannot Read
        ↓
Low-integrity data
```

The reasoning is that reading untrusted or low-integrity information could allow unreliable information to influence a high-integrity process.

## 4.2 No Write Up

The ***-Integrity Axiom**, commonly remembered as **"No Write Up"**, prevents a subject from writing to a higher-integrity object.

For example:

```text
Low-integrity process
        ↓
Cannot Write
        ↓
High-integrity database
```

This prevents a low-trust process or user from modifying highly trusted information.

### BIBA Summary

| Rule             | Meaning                                               |
| ---------------- | ----------------------------------------------------- |
| **No Read Down** | Cannot read information at a lower integrity level    |
| **No Write Up**  | Cannot modify information at a higher integrity level |

### Main Goal

**BIBA = Integrity**

It focuses on preventing **unauthorized or unreliable modification of trusted information**.

---

# 5. Bell-La Padula vs BIBA

The easiest way to remember the difference is:

```text
Bell-La Padula → Confidentiality
BIBA            → Integrity
```

Their rules move in opposite directions:

| Model              | Focus           | Read Rule    | Write Rule    |
| ------------------ | --------------- | ------------ | ------------- |
| **Bell-La Padula** | Confidentiality | No Read Up   | No Write Down |
| **BIBA**           | Integrity       | No Read Down | No Write Up   |

### Easy Memory Trick

Think about **secrets** for Bell-La Padula:

> **Secret information should not move downward.**

Think about **integrity** for BIBA:

> **Untrusted information should not move upward.**

---

# 6. Clark-Wilson Model

The **Clark-Wilson model** is another integrity model, but it approaches integrity differently from BIBA.

BIBA mainly uses **access rules and integrity levels**, while Clark-Wilson focuses on ensuring that **data can only be modified through authorized and well-defined procedures**.

It was designed particularly with **commercial applications and transaction processing** in mind.

Examples include:

* Banking systems
* Accounting systems
* Financial applications
* Business transaction systems

Imagine a banking system where an employee needs to transfer money between two accounts.

Instead of allowing the employee to directly modify the database:

```text
Employee
   ↓
Directly modify database
```

Clark-Wilson encourages:

```text
Employee
   ↓
Authorized transaction
   ↓
Verification
   ↓
Database update
```

This makes it possible to ensure that transactions follow approved rules.

---

## 6.1 Well-Formed Transactions

A **well-formed transaction** is a controlled operation that changes data according to predefined rules.

For example, transferring $100 from Account A to Account B should involve:

```text
Account A: $500
Account B: $200

Transfer $100
      ↓
Account A: $400
Account B: $300
```

The transaction must be performed correctly.

A user should not be allowed to arbitrarily change:

```text
Account A = $500
```

to:

```text
Account A = $50,000
```

Instead, changes must occur through authorized procedures.

---

## 6.2 Separation of Duties

Clark-Wilson also emphasizes **separation of duties**.

The basic idea is:

> One person should not have complete control over a sensitive transaction.

For example, in a financial organization:

```text
Employee A
    ↓
Creates transaction
    ↓
Employee B
    ↓
Approves transaction
```

This reduces the possibility of fraud.

If the same person could both create and approve a transaction, they might create a fraudulent transaction for their own benefit.

---

# 7. Important Clark-Wilson Concepts

Clark-Wilson introduces several important concepts.

### 7.1 Constrained Data Items (CDIs)

**Constrained Data Items (CDIs)** are data objects whose integrity must be protected.

Examples:

* Bank account balances
* Employee salaries
* Financial records
* Inventory records

These objects should only be modified through authorized procedures.

```text
CDI
 ↓
Must be protected
 ↓
Can only be modified through
authorized procedures
```

### 7.2 Unconstrained Data Items (UDIs)

**Unconstrained Data Items (UDIs)** are data items that have not yet been subjected to the system's integrity controls.

For example, raw input received from an external source may initially be treated as a UDI.

```text
External Input
      ↓
     UDI
      ↓
Validation
      ↓
Trusted/controlled data
```

### 7.3 Integrity Verification Procedures (IVPs)

**Integrity Verification Procedures (IVPs)** are procedures used to verify that the system's data remains in a valid and consistent state.

For example, an IVP might verify that:

```text
Total Account Balance
        =
Sum of Individual Transactions
```

If the values do not match, the system can identify a potential integrity problem.

---

# 8. Comparing the Five Models

| Model              | Main Focus           | Key Idea                                     |
| ------------------ | -------------------- | -------------------------------------------- |
| **DAC**            | Access control       | Owner controls access                        |
| **MAC**            | Centralized security | Administrator and labels control access      |
| **Bell-La Padula** | Confidentiality      | Prevent information disclosure               |
| **BIBA**           | Integrity            | Prevent contamination of trusted data        |
| **Clark-Wilson**   | Integrity            | Protect data through controlled transactions |

A simple way to visualize them is:

```text
                    ACCESS CONTROL
                         │
              ┌──────────┴──────────┐
              │                     │
             DAC                   MAC
              │                     │
       Owner controls       Security policy controls
                                  │
                         ┌────────┴────────┐
                         │                 │
                  Confidentiality       Integrity
                         │                 │
                  Bell-La Padula       BIBA
                                           │
                                  Transaction Integrity
                                           │
                                    Clark-Wilson
```

---



> These access control models provide different approaches to protecting information and maintaining system security. DAC emphasizes ownership and flexibility, MAC enforces centrally defined security policies, Bell-La Padula protects confidentiality, BIBA protects integrity, and Clark-Wilson ensures that sensitive data is modified only through controlled and verifiable transactions.
>
> Understanding these models provides a foundation for studying how operating systems, databases, networks, and security frameworks implement access control in real-world environments.


