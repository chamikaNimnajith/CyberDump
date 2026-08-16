# Multilevel Security, Trusted Systems, and Security Evaluation

Modern information systems may need to process information with different levels of sensitivity at the same time. For example, a government system might contain **Unclassified**, **Confidential**, **Secret**, and **Top Secret** information. Different users may have different security clearances and different business needs.

Security mechanisms must therefore ensure that users can access the information they are authorized to use without accidentally or intentionally accessing information at a higher security level.

This leads to the concepts of **Multilevel Security (MLS)**, **Trusted Computing**, and **Security Evaluation**.

---

## 1. Multilevel Security (MLS)

### What Is Multilevel Security?

**Multilevel Security (MLS)** refers to a class of systems that contain resources, particularly information, at **multiple security levels**.

The system allows users with different clearances and need-to-know restrictions to use the same system while preventing unauthorized information flows.

For example:

```text
                    MLS System
                        │
       ┌────────────────┼────────────────┐
       │                │                │
   Top Secret         Secret         Confidential
       │                │                │
   User A             User B           User C
```

Each user is allowed to access information according to their:

* Security clearance
* Assigned permissions
* Need to know
* Role
* Security policies

The objective is to allow legitimate information sharing while preventing unauthorized information disclosure.

---

## 2. MLS and Role-Based Access Control

MLS can be implemented together with **Role-Based Access Control (RBAC)**.

In RBAC, permissions are assigned to **roles**, and users are assigned to those roles.

```text
User
  ↓
Role
  ↓
Permissions
  ↓
Resources
```

For an MLS environment, additional security constraints are required.

For example:

```text
User A
Clearance: Secret
Role: Analyst

User B
Clearance: Confidential
Role: Employee
```

The system must ensure that the permissions associated with each role do not allow users to violate the required security levels.

An MLS implementation based on the **Bell-LaPadula (BLP)** model therefore needs to define:

* Security constraints on users
* Read permissions
* Write permissions
* Role definitions
* User-role assignments
* Security levels associated with resources

The security system then combines **role permissions** with **security-level restrictions**.

---

# 3. MLS Database Security

Multilevel security becomes particularly interesting when applied to databases.

A database may contain information belonging to several security levels:

```text
Database
│
├── Top Secret Data
├── Secret Data
├── Confidential Data
└── Unclassified Data
```

Each row, column, or individual data element may potentially have a security classification.

The database must then enforce rules governing both **reading** and **writing**.

---

# 4. MLS Read Access

The Bell-LaPadula model requires the **simple security property**, commonly expressed as:

> **No Read Up**

A subject cannot read information at a higher security level than the subject's clearance.

For example:

```text
User Clearance: Confidential

Confidential Data    ✓ Read
Secret Data          ✗ Read
Top Secret Data      ✗ Read
```

This prevents a lower-cleared user from directly accessing higher-classified information.

---

## 4.1 Database-Level Enforcement

At a relatively coarse level, enforcing this rule is straightforward.

For example, an entire database or table could be classified as **Secret**.

```text
Secret Database
      ↓
Secret Users → ✓
Confidential Users → ✗
```

The database management system can simply deny access to users without the required clearance.

However, the problem becomes more complicated when security labels are applied to smaller units of information.

---

# 5. Inference Problems in MLS Databases

Suppose individual rows, columns, or elements have different security levels.

A user may not be directly authorized to read restricted information, but carefully constructed queries may reveal information **indirectly**.

This is known as an **inference problem**.

For example:

```text
User:
"How many employees are in Department X?"

Database:
"No result."
```

The response itself may provide information.

The user might infer that:

* The department exists but its information is restricted.
* The query returned no records.
* Certain records have a higher classification.
* A particular person or event may exist.

Therefore, simply hiding the restricted value may not be enough.

Even the **behavior of the database** can potentially reveal information.

---

## 5.1 Granularity and Inference

MLS databases may apply security labels at different granularities:

```text
Database
   ↓
Table
   ↓
Row
   ↓
Column
   ↓
Individual Element
```

The finer the granularity, the more complicated security enforcement can become.

For example:

```text
Table-level security
       ↓
Relatively simple

Row-level security
       ↓
More complex

Element-level security
       ↓
Potentially significant inference problems
```

This is one of the major challenges of implementing MLS in database systems.

---

# 6. MLS Write Access

Bell-LaPadula also defines the ***-security property**, commonly called:

> **No Write Down**

A subject should not write information from a higher security level into a lower security level.

For example:

```text
Secret User
     │
     ├── Write to Secret      ✓
     │
     └── Write to Confidential ✗
```

This prevents sensitive information from flowing downward to a lower security level.

However, database operations can create interesting conflicts.

---

# 7. The Primary-Key Conflict Problem

Consider a database containing employee records.

Suppose a high-level record already exists:

```text
Security Level: SECRET

Employee ID: 1001
Name: Alice
Salary: ...
```

Now a lower-level user attempts to insert:

```text
Security Level: CONFIDENTIAL

Employee ID: 1001
Name: Alice
```

The database sees that the primary key `1001` already exists.

But the lower-level user should not be able to see the Secret record.

This creates a difficult security problem.

The database has several possible choices.

---

# 8. Alternative 1 — Rejection

The database could reject the insertion.

```text
INSERT Employee ID 1001
          ↓
Primary key already exists
          ↓
       REJECT
```

The problem is that the user may now infer that a record with that key already exists.

For example:

```text
User:
"Why was my insertion rejected?"

Database:
"Primary key already exists."
```

The response potentially reveals the existence of the higher-level record.

Therefore, **rejection can create an information leakage problem**.

---

# 9. Alternative 2 — Replacement

Another possibility is to replace the existing row.

```text
Existing Secret Row
        ↓
     Replace
        ↓
New Confidential Row
```

This is dangerous because the lower-level user could effectively overwrite higher-level information.

That violates the intended security and integrity properties of the system.

Therefore, simple replacement is generally unacceptable in a properly designed MLS database.

---

# 10. Alternative 3 — Polyinstantiation

A third approach is **polyinstantiation**.

Polyinstantiation allows multiple records to exist with the same apparent key but at different security levels.

For example:

```text
Employee ID: 1001
Security Level: SECRET
Data: Alice — Secret Information

Employee ID: 1001
Security Level: CONFIDENTIAL
Data: Alice — Public/Confidential Information
```

The two users can therefore see different versions of what appears to be the same record.

```text
Secret User
     ↓
Secret Version

Confidential User
     ↓
Confidential Version
```

This prevents the lower-level user from learning that the higher-level record exists.

However, polyinstantiation introduces additional complexity because the database now contains multiple versions of information associated with the same key.

---

# 11. Trusted Platform Module (TPM)

A **Trusted Platform Module (TPM)** is a hardware-based security component designed to establish a **root of trust** for a computing system.

The TPM concept was developed through the **Trusted Computing Group (TCG)**.

A TPM can be implemented as a dedicated hardware component associated with a computer's:

* Motherboard
* Processor platform
* Smart-card-like hardware

It provides protected functionality for cryptographic operations and keys.

Conceptually:

```text
                 TPM
                  │
        ┌─────────┼─────────┐
        │         │         │
     Secure    Keys      Platform
      Boot              Integrity
```

The TPM helps establish trust starting from hardware and extending upward into the software environment.

---

# 12. TPM Core Services

Three important TPM capabilities are:

1. Authenticated boot
2. Certification
3. Encryption

---

## 12.1 Authenticated Boot

Authenticated boot establishes a chain of trust during system startup.

Instead of simply loading software, the system verifies the integrity or authenticity of components before allowing the next stage to execute.

A simplified sequence is:

```text
Hardware
   ↓
TPM
   ↓
Boot Code
   ↓
Bootloader
   ↓
Operating System
   ↓
Applications
```

Each stage can verify the next stage.

The TPM can also maintain a **tamper-evident record** of the software components involved in the boot process.

This creates a **chain of trust**.

If an unauthorized modification occurs, the recorded measurements may no longer match the expected configuration.

---

# 13. Certification Service

The TPM can also provide a mechanism for **certifying the configuration of a system** to an external party.

A simplified process is:

```text
TPM
 ↓
Platform Measurements
 ↓
Digital Certification
 ↓
External Party
```

This allows another system to obtain evidence about the state of the platform.

Trust can therefore extend hierarchically:

```text
TPM
 ↓
Operating System
 ↓
Applications
```

The underlying idea is that if the platform's trusted foundation can be verified, confidence can be extended to higher-level software.

---

# 14. TPM Encryption Service

TPMs can also support cryptographic operations and protect encryption keys.

A machine-specific secret can be associated with the platform so that protected information is intended to be accessible only under appropriate conditions.

Conceptually:

```text
Data
 ↓
Encryption
 ↓
Machine-Specific Key
 ↓
Protected Storage
```

When the system is operating in the expected trusted configuration, the necessary cryptographic protection can be made available.

This can help protect information even if an attacker obtains access to the physical storage device.

---

# 15. The History of Trusted Systems

The concept of trusted computing systems has a long history.

Research into trusted systems began in the **early 1970s**, particularly in environments where highly sensitive government and military information needed to be protected.

These efforts eventually contributed to formal security evaluation criteria.

---

## 15.1 Trusted Computer System Evaluation Criteria

In the early 1980s, the United States developed the **Trusted Computer System Evaluation Criteria (TCSEC)**.

TCSEC became widely known as the **Orange Book**.

Its purpose was to provide a structured way to evaluate the security capabilities of computer systems.

Security properties and evaluation levels were used to distinguish systems according to their security capabilities and assurance.

---

# 16. NSA and Trusted Product Evaluation

The **National Security Agency (NSA)** played an important role in the evaluation of trusted computing products.

Its Computer Security Center operated programs for evaluating commercially available computer products for potential use in Defense environments.

This helped establish the concept that security products should not simply claim to be secure; their security properties should be **examined and evaluated systematically**.

---

# 17. From TCSEC to Common Criteria

As computing became increasingly international, different countries and regions developed their own security evaluation approaches.

This created a problem:

```text
Country A → Evaluation System A
Country B → Evaluation System B
Country C → Evaluation System C
```

A common international framework was needed.

This led to the development of the **Common Criteria (CC)** in the 1990s.

Common Criteria became an international framework for specifying and evaluating the security properties of IT products.

---

# 18. Common Criteria

The **Common Criteria** is an international framework used to define security requirements and evaluate IT products.

It provides a structured way for different stakeholders to gain confidence that a product's claimed security properties have been properly evaluated.

The framework separates security requirements into two broad categories:

* **Functional requirements**
* **Assurance requirements**

---

## 18.1 Functional Requirements

Functional requirements describe **what security functions the product should provide**.

Examples can include requirements related to:

* Access control
* Identification and authentication
* Security auditing
* Cryptographic support
* Protection of user data

In simple terms:

> **Functional requirements describe what the security system must do.**

---

## 18.2 Assurance Requirements

Assurance requirements concern the degree of confidence that the security functions have been correctly implemented.

They examine things such as:

* Design
* Development
* Testing
* Documentation
* Verification
* Vulnerability analysis

In simple terms:

> **Assurance requirements ask how confidently we can trust that the security functions actually work as intended.**

---

# 19. Target of Evaluation (TOE)

The **Target of Evaluation (TOE)** is the specific product or system being evaluated.

For example, a TOE could be:

* An operating system
* A firewall
* A smart card
* A database system
* A network security product

Conceptually:

```text
Product/System
      ↓
Target of Evaluation
      ↓
Security Evaluation
```

The evaluation focuses on the defined TOE and its relevant security functionality.

---

# 20. Protection Profiles

A **Protection Profile (PP)** defines security requirements for a particular category of products.

Instead of creating completely new requirements for every product, a Protection Profile establishes a common set of security expectations.

For example:

```text
Protection Profile
        ↓
Smart Card Products
        ↓
Common Security Requirements
```

A Smartcard Protection Profile may define:

* Functional security requirements
* Assurance requirements
* Relevant threats
* Security objectives

A PP therefore describes **what a category of products is expected to provide**.

---

# 21. Security Target

A **Security Target (ST)** is specific to the product being evaluated.

It maps the relevant security requirements to the particular **Target of Evaluation (TOE)**.

The relationship can be viewed as:

```text
Protection Profile
        ↓
General Security Requirements
        ↓
Security Target
        ↓
Specific Product / TOE
```

For example:

```text
PP:
Requirements for Smart Cards

        ↓

ST:
Security Target for Smart Card X

        ↓

TOE:
Smart Card X
```

The Security Target explains how the specific product addresses the applicable security requirements.

---

# 22. Security Assurance

**Security assurance** refers to the degree of confidence that security controls:

* Operate correctly
* Have been properly implemented
* Protect the system as intended

Assurance is different from simply having a security feature.

For example, a product may claim:

> "This product provides secure authentication."

Assurance asks:

> "What evidence demonstrates that the authentication mechanism has been correctly designed, implemented, and tested?"

This distinction is important in high-security environments.

---

# 23. Evaluation Assurance Levels

Common Criteria defines **seven Evaluation Assurance Levels (EALs)**.

They represent increasing levels of evaluation rigor.

| Level     | Description                                 |
| --------- | ------------------------------------------- |
| **EAL 1** | Functionally tested                         |
| **EAL 2** | Structurally tested                         |
| **EAL 3** | Methodically tested and checked             |
| **EAL 4** | Methodically designed, tested, and reviewed |
| **EAL 5** | Semiformally designed and tested            |
| **EAL 6** | Semiformally verified design and tested     |
| **EAL 7** | Formally verified design and tested         |

The general progression is:

```text
EAL 1
  ↓
EAL 2
  ↓
EAL 3
  ↓
EAL 4
  ↓
EAL 5
  ↓
EAL 6
  ↓
EAL 7
```

Higher levels generally require more rigorous evaluation and therefore involve greater effort and cost.

---

# 24. Evaluating a Security Product

A security evaluation does not focus only on the product's user interface.

Evidence can be examined at multiple levels, including:

```text
Security Target
      ↓
Architecture / Design
      ↓
Specifications
      ↓
Implementation
      ↓
Source Code
      ↓
Hardware
```

The evaluator examines the appropriate evidence to determine whether the TOE satisfies its defined security requirements.

---

# 25. Parties Involved in Evaluation

A Common Criteria evaluation can involve several different parties.

## Sponsor

The **sponsor** is typically the customer or vendor requesting the evaluation.

The sponsor provides the product and initiates or supports the evaluation process.

---

## Developer

The **developer** develops the TOE and provides the technical evidence required for evaluation.

This may include:

* Design documentation
* Specifications
* Testing information
* Source-code evidence
* Security documentation

---

## Evaluator

The **evaluator** independently examines the evidence and performs the required evaluation activities.

The evaluator determines whether the product satisfies the defined requirements.

---

## Certifier

The **certifier** provides oversight of the evaluation process and helps ensure that the evaluation is conducted according to the applicable scheme and requirements.

The overall relationship can be illustrated as:

```text
Sponsor
   │
   │ Requests Evaluation
   ▼
Developer
   │
   │ Provides TOE and Evidence
   ▼
Evaluator
   │
   │ Performs Evaluation
   ▼
Certifier
   │
   │ Oversees / Confirms Evaluation
   ▼
Evaluation Result
```

---

# 26. Evaluation Process

A security evaluation can be divided into three broad phases.

## Preparation

The evaluation scope and requirements are established.

Activities may include:

* Defining the TOE
* Preparing the Security Target
* Identifying applicable requirements
* Preparing evaluation evidence
* Establishing the evaluation plan

---

## Conduct of Evaluation

The evaluator examines the TOE and supporting evidence.

Activities may include:

* Reviewing design documentation
* Examining specifications
* Reviewing source-code evidence
* Testing security functionality
* Performing vulnerability analysis
* Checking assurance requirements

The depth of these activities depends on the applicable evaluation requirements.

---

## Conclusion

The final phase involves completing the evaluation and producing the appropriate results.

The process can therefore be represented as:

```text
Preparation
     ↓
Evaluation
     ↓
Conclusion
     ↓
Evaluation Result
```

---

# 27. Connecting MLS, Trusted Computing, and Evaluation

These concepts address different aspects of system security but are closely related.

**Multilevel Security** focuses on controlling information according to security levels:

```text
Who can access what?
```

**Trusted Computing and TPMs** focus on establishing a trustworthy foundation for the computing platform:

```text
Can we trust the platform and its configuration?
```

**Common Criteria** focuses on systematically evaluating security functionality and assurance:

```text
How much confidence do we have in the security claims?
```

Together, they illustrate an important principle of security engineering:

```text
Access Control
      +
Trusted Platform
      +
Security Evaluation
      ↓
Greater Confidence in System Security
```

A secure system therefore requires more than individual security mechanisms. It needs carefully defined access rules, a trustworthy computing foundation, and evidence-based evaluation of the controls that are intended to protect the system.
