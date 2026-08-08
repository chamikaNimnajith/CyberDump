# Active Directory (AD)

Active Directory (AD) is Microsoft's centralized system for managing **users, computers, devices, permissions, and security policies** in a Windows-based network.

It is commonly used in organizations where hundreds or thousands of computers and users need to be managed from a central location.

Instead of configuring every computer individually, administrators can use Active Directory to centrally manage:

* 👤 User accounts
* 💻 Computers
* 👥 Groups
* 🔐 Permissions
* 🛡️ Security policies
* ⚙️ Configuration settings
* 📜 Certificates
* 🌐 Network resources

A simple way to think about Active Directory is:

> **Active Directory is a centralized database and management system that allows an organization to control who can access what, and how computers in the network should behave.**

---

## 1. What Does Active Directory Do?

Active Directory provides several important services.

### Authentication

**Authentication** answers:

> "Who are you?"

When a user logs into a domain-joined Windows computer, Active Directory verifies the user's identity.

For example:

```text
Username: chamika
Password: ********
        ↓
Domain Controller
        ↓
Is this user valid?
        ↓
Yes → Authentication successful
```

Authentication can involve passwords, certificates, and other mechanisms depending on the environment.

---

### Authorization

**Authorization** answers:

> "What are you allowed to access?"

After a user has been authenticated, Active Directory and Windows security mechanisms determine what that user is allowed to do.

For example:

```text
User: Alice

Can access:
✓ Company Documents
✓ Finance Share

Cannot access:
✗ Domain Controller
✗ HR Documents
```

Authentication and authorization are therefore different:

| Concept        | Question                        |
| -------------- | ------------------------------- |
| Authentication | Who are you?                    |
| Authorization  | What are you allowed to access? |

---

### Directory Services

Active Directory acts as a centralized directory containing information about objects in the organization.

These objects can include:

* Users
* Computers
* Groups
* Printers
* Servers
* Organizational Units
* Security policies

Administrators can search and manage these objects from a central location.

---

### Active Directory Certificate Services (AD CS)

**Active Directory Certificate Services (AD CS)** allows an organization to operate its own certificate infrastructure.

It can be used to issue and manage **X.509 certificates** for users, computers, servers, and other services.

For example:

```text
Active Directory
       │
       └── AD CS
             │
             ├── User certificates
             ├── Computer certificates
             └── Server certificates
```

Certificates can be used for authentication, encryption, digital signatures, and other security purposes.

---

# 2. Core Components of Active Directory

Several components work together to make an Active Directory environment function.

---

## 2.1 Domain Controllers (DCs)

A **Domain Controller (DC)** is a Windows Server that runs Active Directory Domain Services (AD DS).

Domain Controllers are one of the most important components of an AD environment.

They are responsible for tasks such as:

* Storing Active Directory information
* Authenticating users
* Processing directory queries
* Applying security policies
* Managing domain objects
* Participating in AD replication

A simplified login process looks like this:

```text
User
 │
 │ Username + Password
 ↓
Domain Controller
 │
 ├── Verify credentials
 ├── Check account status
 └── Determine group membership
 │
 ↓
Authentication Result
```

### Multiple Domain Controllers

Organizations normally deploy multiple Domain Controllers.

For example:

```text
             Domain
                │
       ┌────────┴────────┐
       ↓                 ↓
     DC01               DC02
       │                 │
       └──── Replication ─┘
```

The Domain Controllers replicate directory information between each other.

This provides:

* **Redundancy** — another DC can continue providing services if one fails.
* **Availability** — authentication services remain available.
* **Load distribution** — requests can be handled by multiple DCs.

---

# 2.2 Domain Admins and Enterprise Admins

Active Directory contains privileged groups that provide extensive administrative control.

### Domain Admins

The **Domain Admins** group has highly privileged administrative access within a specific domain.

Members can generally perform tasks such as:

* Managing users and groups
* Managing computers
* Changing domain configuration
* Managing Group Policies
* Administering Domain Controllers

> Domain Admin is extremely powerful and should be assigned only when necessary.

---

### Enterprise Admins

**Enterprise Admins** is a highly privileged group that operates at the **forest level**.

It provides administrative control across the Active Directory forest.

A simplified hierarchy is:

```text
Forest
  │
  ├── Domain A
  │     └── Domain Admins
  │
  └── Domain B
        └── Domain Admins

Enterprise Admins
        ↓
Forest-wide administrative control
```

### Domain vs Enterprise Administration

| Group             | Scope           |
| ----------------- | --------------- |
| Domain Admins     | Specific domain |
| Enterprise Admins | Entire forest   |

These groups should be carefully controlled because compromise of highly privileged accounts can have a major impact on the entire AD environment.

---

## Local vs Domain Groups

Windows has both **local** and **domain-level** accounts and groups.

For example:

```text
Local Computer
      │
      └── Local Administrators

Active Directory Domain
      │
      └── Domain Admins
```

A user may have administrative privileges on one computer without being a Domain Admin.

Useful commands include:

```cmd
net user
```

Lists local user accounts.

```cmd
net user /domain
```

Queries domain user accounts.

```cmd
net group
```

Lists local groups.

```cmd
net group /domain
```

Queries domain groups.

```cmd
whoami /groups
```

Displays the groups associated with the current user.

---

# 2.3 Group Policy Objects (GPOs)

**Group Policy Objects (GPOs)** allow administrators to centrally configure Windows computers and users.

Instead of configuring 500 computers individually, an administrator can create one policy and apply it to the required computers or users.

For example:

```text
                Domain
                  │
               GPO
                  │
        ┌─────────┼─────────┐
        ↓         ↓         ↓
       PC01      PC02      PC03
```

The same configuration can therefore be applied consistently across many systems.

### Examples of GPO Settings

GPOs can control settings such as:

* Password policies
* Account lockout policies
* Windows Firewall settings
* Software restrictions
* User rights
* Windows Update configuration
* Login scripts
* Desktop settings
* Security configurations

For example, an administrator could configure a policy requiring:

```text
Minimum password length: 14
Account lockout threshold: 5 attempts
```

---

## Managing GPOs

Group Policy can be managed using:

```text
gpmc.msc
```

This opens the **Group Policy Management Console**.

GPOs can be associated with:

* Sites
* Domains
* Organizational Units (OUs)

Security filtering can also be used to control which users or computers receive a particular policy.

---

## GPO Refresh

Group Policy is periodically refreshed automatically.

Administrators can also force an immediate refresh using:

```cmd
gpupdate /force
```

This tells Windows to reapply applicable computer and user policies.

---

# 2.4 Organizational Units (OUs)

An **Organizational Unit (OU)** is a container used to organize objects inside an Active Directory domain.

Objects such as users, computers, and groups can be placed inside OUs.

For example:

```text
Company.local
│
├── IT
│   ├── Users
│   └── Computers
│
├── HR
│   ├── Users
│   └── Computers
│
└── Finance
    ├── Users
    └── Computers
```

This allows organizations to organize AD according to their business structure.

---

## Why Are OUs Important?

OUs are particularly useful for applying policies.

For example:

```text
Finance OU
     │
     └── Finance computers
              │
              └── Finance GPO
```

A GPO can be linked to the Finance OU so that computers or users within that OU receive the appropriate settings.

OUs also help administrators implement:

* Administrative delegation
* Least privilege
* Policy management
* Organizational structure

---

## Managing OUs

The **Active Directory Users and Computers (ADUC)** console can be opened using:

```text
dsa.msc
```

Administrators can use it to manage:

* Users
* Groups
* Computers
* OUs
* Other AD objects

---

# 2.5 LDAP

**LDAP (Lightweight Directory Access Protocol)** is a protocol used to communicate with directory services.

In Active Directory, LDAP allows applications and administrators to query directory information.

For example, an administrator may want to find:

```text
All users in the IT department
```

LDAP can be used to query the directory and retrieve that information.

PowerShell provides commands such as:

```powershell
Get-ADUser
```

For example:

```powershell
Get-ADUser -Filter *
```

can retrieve user objects from Active Directory.

A simple way to remember LDAP is:

> **LDAP is a protocol used to communicate with and query directory information.**

---

# 2.6 Kerberos

**Kerberos** is the primary authentication protocol used by modern Active Directory environments.

Instead of repeatedly sending a user's password across the network, Kerberos uses **tickets** to prove authentication.

A simplified process looks like this:

```text
User
 │
 │ Authentication request
 ↓
Domain Controller / KDC
 │
 │ Issues ticket
 ↓
Kerberos Ticket
 │
 ↓
Network Service
```

---

## Key Distribution Center (KDC)

The **Key Distribution Center (KDC)** is responsible for Kerberos authentication and ticket management.

In an Active Directory environment, the KDC runs as a service on the Domain Controller.

It contains two major logical components:

* **Authentication Service (AS)**
* **Ticket Granting Service (TGS)**

A simplified flow is:

```text
User
  │
  ↓
Authentication Service
  │
  ↓
Ticket Granting Ticket (TGT)
  │
  ↓
Ticket Granting Service
  │
  ↓
Service Ticket
  │
  ↓
Network Service
```

---

## Viewing Kerberos Tickets

Windows provides the `klist` command for viewing Kerberos tickets associated with the current logon session.

For example:

```cmd
klist
```

or:

```cmd
klist tickets
```

---

> **Security Note:** Terms such as **Golden Ticket** and **Silver Ticket** refer to forged Kerberos tickets used in specific attack scenarios. They are important concepts in Active Directory security and penetration testing.

---

# 2.7 NTDS.dit

One of the most important files on a Domain Controller is:

```text
C:\Windows\NTDS\ntds.dit
```

`ntds.dit` is the Active Directory database.

It stores information about the domain, including objects such as:

* Users
* Groups
* Computers
* Security-related information
* Directory metadata

It also contains password-derived credential data, including password hashes.

Because of its sensitivity, access to `ntds.dit` must be strongly protected.

> **Security Note:** Unauthorized access to `ntds.dit` can expose highly sensitive domain credential information and may lead to serious domain compromise.

---

# 2.8 SYSVOL

**SYSVOL (System Volume)** is a shared directory found on Domain Controllers.

It stores important domain-wide files such as:

* Group Policy files
* Login scripts
* Other files required for domain-wide configuration

SYSVOL is replicated between Domain Controllers.

For example:

```text
             Domain
                │
        ┌───────┴───────┐
        ↓               ↓
       DC01            DC02
        │               │
      SYSVOL  ←──────→ SYSVOL
```

Historically, SYSVOL replication could use **FRS (File Replication Service)**. Modern environments generally use **DFSR (Distributed File System Replication)**.

---

# 2.9 Global Catalog (GC)

The **Global Catalog (GC)** is a distributed database containing information about objects throughout the Active Directory forest.

Unlike a normal domain directory, the Global Catalog contains a **partial replica of objects from every domain in the forest**.

This makes it possible to perform searches across domains efficiently.

For example:

```text
Forest
│
├── Domain A
│   ├── Users
│   └── Computers
│
└── Domain B
    ├── Users
    └── Computers

        ↓

   Global Catalog
        │
        └── Partial information
            from both domains
```

---

# 3. Active Directory Hierarchy

Active Directory can grow from a single domain into a large multi-domain environment.

The main organizational levels are:

```text
Forest
  │
  ├── Tree
  │    │
  │    ├── Domain
  │    └── Domain
  │
  └── Tree
       │
       ├── Domain
       └── Domain
```

Let's understand each level.

---

# 3.1 Domain

A **domain** is a fundamental administrative unit in Active Directory.

It contains objects such as:

* Users
* Computers
* Groups
* Domain Controllers
* OUs

For example:

```text
example.local
│
├── Users
├── Computers
├── Groups
└── Domain Controllers
```

A domain normally has its own Domain Controllers and security boundary.

---

# 3.2 Tree

An AD **tree** is a collection of one or more domains that share a **contiguous DNS namespace**.

For example:

```text
example.local
│
├── sales.example.local
├── hr.example.local
└── it.example.local
```

All these domains belong to the same namespace hierarchy.

The root domain is:

```text
example.local
```

and the child domains are:

```text
sales.example.local
hr.example.local
it.example.local
```

Together they form an AD tree.

---

# 3.3 Forest

A **forest** is the highest-level logical structure in Active Directory.

A forest can contain:

* Multiple domains
* Multiple domain trees
* Multiple Domain Controllers
* A shared schema
* A Global Catalog

Different trees within the same forest do **not** have to use the same DNS namespace.

For example:

```text
                 Forest
                   │
        ┌──────────┴──────────┐
        ↓                     ↓
      Tree 1                Tree 2
        │                     │
   example.com             example.org
        │                     │
   ┌────┴────┐          ┌─────┴─────┐
   ↓         ↓          ↓           ↓
 sales     hr         asia        europe
```

The entire structure is considered one Active Directory forest.

---

# 3.4 Trust Relationships

A **trust relationship** allows users and resources from different AD domains or forests to interact under defined security rules.

For example:

```text
Domain A
   │
   │ Trust
   ↓
Domain B
```

A user from Domain A may be able to access a resource in Domain B if the appropriate permissions are granted.

Trusts are therefore useful when organizations need to allow controlled access between different domains or forests.

### Important Point

A trust does **not automatically mean** that every user gets access to everything.

Instead:

```text
Trust
  +
User authentication
  +
Resource permissions
  =
Actual access
```

The trust makes cross-domain authentication possible, while permissions determine what the authenticated user can actually access.

---

# 4. Putting Everything Together

The different Active Directory components work together as one system.

Consider a company with hundreds of employees.

```text
                         Active Directory
                               │
                         ┌─────┴─────┐
                         │  Domain   │
                         └─────┬─────┘
                               │
          ┌────────────────────┼────────────────────┐
          ↓                    ↓                    ↓
   Domain Controllers         OUs                  GPOs
          │                    │                    │
          │              ┌─────┼─────┐              │
          │              ↓     ↓     ↓              │
          │             IT    HR   Finance           │
          │                                           │
          └──────────── Authentication ───────────────┘
                               │
                         Kerberos / LDAP
                               │
                               ↓
                         Users & Computers
```

For example, when an employee logs into a domain-joined computer:

1. The user provides their credentials.
2. The computer communicates with a Domain Controller.
3. Kerberos is used for authentication.
4. Active Directory determines the user's identity and group memberships.
5. Group Policy settings are applied.
6. Windows determines which resources the user is authorized to access.
7. The user receives access according to their permissions.

---

# 5. Important Active Directory Components at a Glance

| Component             | Purpose                                                   |
| --------------------- | --------------------------------------------------------- |
| **Domain Controller** | Authenticates users and manages the domain                |
| **Active Directory**  | Central directory and management system                   |
| **Domain**            | Fundamental administrative unit                           |
| **Tree**              | Collection of domains sharing a contiguous namespace      |
| **Forest**            | Collection of domains/trees sharing AD infrastructure     |
| **OU**                | Organizes AD objects                                      |
| **GPO**               | Centrally applies configuration and security policies     |
| **LDAP**              | Protocol for querying and managing directory information  |
| **Kerberos**          | Primary AD authentication protocol                        |
| **KDC**               | Provides Kerberos authentication and tickets              |
| **NTDS.dit**          | Main AD database                                          |
| **SYSVOL**            | Stores replicated GPO files and scripts                   |
| **Global Catalog**    | Provides partial directory information across the forest  |
| **Domain Admins**     | Highly privileged administrators within a domain          |
| **Enterprise Admins** | Highly privileged administrators across a forest          |
| **Trust**             | Enables controlled authentication between domains/forests |

---

# 6. Simple Mental Model

If Active Directory seems complicated at first, remember this simplified model:

```text
                 ACTIVE DIRECTORY
                        │
        ┌───────────────┼────────────────┐
        ↓               ↓                ↓
     USERS          COMPUTERS         GROUPS
        │               │                │
        └───────────────┼────────────────┘
                        ↓
                       OUs
                        │
                        ↓
                       GPOs
                        │
                        ↓
              Centralized Management
                        │
             ┌──────────┴──────────┐
             ↓                     ↓
          Kerberos               LDAP
       Authentication       Directory Queries
             │                     │
             └──────────┬──────────┘
                        ↓
               Domain Controllers
                        │
              ┌─────────┴─────────┐
              ↓                   ↓
          NTDS.dit              SYSVOL
```

### In one sentence:

> **Active Directory is Microsoft's centralized identity and directory management system that allows organizations to authenticate users, authorize access, organize network resources, and centrally enforce security and configuration policies.**
