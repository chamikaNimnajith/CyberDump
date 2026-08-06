# Understanding Windows Permissions and Security Model

## Introduction

Windows permissions are a fundamental part of the Windows security architecture. They control:

* Who can access the system
* What resources users can access
* What actions users are allowed to perform
* How Windows protects sensitive files and processes

To understand Windows permissions, we need to understand concepts such as **authentication, authorization, security identifiers, access tokens, ACLs, integrity levels, and User Account Control (UAC).**

---

# 1. Authentication, Authorization, and Session Management

Almost every secure system uses three important security concepts:

1. Authentication
2. Session Management
3. Authorization

These concepts work together to control access to resources.

---

# Authentication — "Who are you?"

Authentication is the process of proving your identity to a system.

Before Windows allows a user to access resources, it must verify who the user is.

Examples of authentication methods:

* Passwords
* PINs
* Fingerprints
* Facial recognition
* Smart cards

Example:

When you log into Windows:

```
Username + Password
          |
          ↓
Windows verifies identity
          |
          ↓
User is authenticated
```

After successful authentication, Windows creates a security context for that user.

---

# Session Management — "Maintaining your access"

After authentication, the system creates a **session**.

A session stores information about the authenticated user so the user does not need to repeatedly enter credentials.

For example:

1. User logs into Windows.
2. Windows creates a session.
3. User opens files, applications, and folders.
4. Windows uses session information to verify permissions.

Without session management, users would need to authenticate every time they open a file or perform an action.

---

# Authorization — "What can you do?"

Authorization determines what actions an authenticated user is allowed to perform.

Authentication answers:

> "Who are you?"

Authorization answers:

> "What are you allowed to do?"

Example:

A user may be authenticated successfully but still cannot:

* Delete system files
* Access another user's private folder
* Install software

because they do not have the required permissions.

Windows performs authorization checks whenever a user accesses a resource.

Example:

```
User opens file
       |
       ↓
Windows checks permissions
       |
       ↓
Allow or Deny access
```

---

# 2. Security Principals and Security Identifiers (SIDs)

## What is a Security Principal?

A **security principal** is any entity that Windows can authenticate and assign permissions to.

Examples:

* User accounts
* Groups
* Computer accounts
* Service accounts

Examples:

```
John
Administrators Group
Domain Users
WORKSTATION01 Computer Account
```

Every security principal has a unique identifier called a **SID**.

---

# What is a SID?

SID stands for:

**Security Identifier**

A SID uniquely identifies an account or group inside Windows.

Example:

```
S-1-5-21-3623811015-3361044348-30300820-1001
```

Windows does not internally use usernames like:

```
Administrator
John
Guest
```

Instead, Windows uses SIDs.

The username is only a friendly label.

---

# SID Structure

A SID contains multiple parts:

Example:

```
S-1-5-21-3623811015-3361044348-30300820-1001
```

---

## S

Indicates that this value is a Security Identifier.

Example:

```
S
```

---

## Revision Number

The second value represents the SID version.

Example:

```
S-1
```

Currently, almost all SIDs use revision number:

```
1
```

---

## Identifier Authority

The next value identifies who created the SID.

Example:

```
S-1-5
```

Common values:

* NT Authority = 5
* Local Security Authority (LSA)
* Domain Controller (for domain accounts)

---

## Domain Identifier

The next section identifies the domain or computer.

Example:

```
S-1-5-21-3623811015-3361044348-30300820
```

Every domain has a unique identifier.

This prevents conflicts in enterprise networks.

---

## Relative Identifier (RID)

The final number identifies the specific user or group.

Example:

```
S-1-5-21-xxx-xxx-xxx-1001
                              |
                              RID
```

Examples:

| RID   | Account               |
| ----- | --------------------- |
| 500   | Administrator         |
| 501   | Guest                 |
| 1000+ | User-created accounts |
| 512   | Domain Admins         |

The RID alone is not unique.

The complete SID is unique because it combines:

```
Domain Identifier + RID
```

---

# Well-Known SIDs

Windows has predefined SIDs created automatically.

Examples:

* Administrators
* Backup Operators
* Users
* Replicators

Windows converts these numbers into readable names.

Example:

Instead of showing:

```
S-1-5-32-544
```

Windows displays:

```
Administrators
```

---

# Viewing SIDs

## Current User SID

Command:

```cmd
whoami /user
```

Example output:

```
User Name        SID
Administrator    S-1-5-21-...-500
```

---

## Display All User SIDs

Command:

```cmd
wmic useraccount get domain,name,sid
```

Example:

```
DOMAIN     USER       SID

PC01       Admin      S-1-5-21-...-500
PC01       John       S-1-5-21-...-1001
```

---

# 3. Access Tokens

## What is an Access Token?

An access token is Windows' way of storing a user's security information during a session.

After authentication:

```
User Login
     |
     ↓
Windows Creates Access Token
     |
     ↓
Token Used for Permission Checks
```

Every process running in Windows has an access token.

---

# Information Stored in an Access Token

An access token contains:

* User SID
* Group SIDs
* User privileges
* Logon information
* Owner SID

Example:

A user belongs to:

```
John
 |
 +-- Users
 |
 +-- Remote Desktop Users
 |
 +-- Administrators
```

The access token contains all these group memberships.

---

# Types of Access Tokens

## Primary Token

A primary token is assigned to a process when it starts.

It defines the default permissions of that process.

Example:

Opening Notepad:

```
User
 |
 ↓
Notepad Process
 |
 ↓
Primary Token
```

---

## Impersonation Token

An impersonation token allows a process to temporarily act as another user.

Example:

A service running as SYSTEM may temporarily impersonate another user.

Attackers often target impersonation tokens because they may allow privilege escalation.

A famous example is abusing:

```
SeImpersonatePrivilege
```

---

# 4. File Permissions (ACLs and ACEs)

Windows controls file access using:

* ACLs
* ACEs

---

# Access Control List (ACL)

An ACL is a list that defines:

* Who can access a file
* What permissions they have

Example:

```
File: secret.txt

ACL:
Administrator → Full Control
John          → Read
Users         → No Access
```

---

# Access Control Entry (ACE)

An ACE is a single permission rule inside an ACL.

Example:

```
John → Read Permission
```

is one ACE.

A file's ACL contains multiple ACEs.

---

# Viewing Permissions

Command:

```cmd
icacls filename
```

Example:

```
secret.txt

Administrator:(F)
John:(R)
Users:(RX)
```

Meaning:

| Permission | Meaning          |
| ---------- | ---------------- |
| F          | Full Control     |
| M          | Modify           |
| R          | Read             |
| W          | Write            |
| RX         | Read and Execute |

---

# Why ACLs Matter in Security

Incorrect permissions can allow privilege escalation.

Example:

A Windows service runs as SYSTEM:

```
Service.exe
     |
     ↓
SYSTEM privileges
```

If a normal user has:

```
Write permission
```

on that executable:

```
User replaces service.exe
        |
        ↓
Service runs malicious code as SYSTEM
```

This is called **service hijacking**.

---

# Modifying Permissions

Grant permission:

```cmd
icacls file.txt /grant username:F
```

Grant recursively:

```cmd
icacls folder /grant username:F /T
```

Continue even with errors:

```cmd
/C
```

---

# PowerShell Permission Commands

View permissions:

```powershell
Get-Acl filename
```

Modify permissions:

```powershell
Set-Acl
```

---

# 5. Mandatory Integrity Control (MIC)

ACLs are not the only security mechanism in Windows.

Windows also uses:

**Mandatory Integrity Control (MIC)**

MIC adds another security layer above normal permissions.

---

# Integrity Levels

Windows has four main integrity levels:

| Level  | Usage                   |
| ------ | ----------------------- |
| System | Core Windows processes  |
| High   | Administrator processes |
| Medium | Normal user processes   |
| Low    | Restricted processes    |

Example:

Normal user:

```
Medium Integrity
```

Administrator:

```
High Integrity
```

---

# How Integrity Levels Work

A process cannot normally access objects with a higher integrity level.

Example:

A Medium integrity application:

```
Chrome
(Medium)
```

cannot modify:

```
System file
(High)
```

even if ACL permissions allow it.

---

# Process Integrity Rules

When a process starts:

```
Process Integrity =
Lowest(User Integrity, File Integrity)
```

Example:

Administrator opens a Low integrity file:

```
Admin = High
File = Low

Process = Low
```

The process will not become High automatically.

---

# Checking Integrity Level

Command:

```cmd
whoami /groups
```

Example:

```
Mandatory Label\High Mandatory Level
```

---

# Changing Integrity Level

Example:

```cmd
icacls file.txt /setintegritylevel high
```

---

# 6. User Account Control (UAC)

## What is UAC?

User Account Control is a Windows security feature that follows the principle of:

**Least Privilege**

Meaning:

> Users should only have the permissions they need.

---

# Why UAC Exists

Without UAC:

```
Administrator User
        |
        ↓
Every program gets admin privileges
```

A malware program could immediately control the system.

With UAC:

```
Administrator User
        |
        ↓
Normal privileges by default
        |
        ↓
Ask permission for admin actions
```

---

# Dual Access Tokens

Administrator accounts actually receive two tokens after login.

## 1. Standard User Token

Used for normal activities:

* Browsing
* Editing documents
* Running applications

---

## 2. Administrator Token

Used for privileged operations.

Examples:

* Installing drivers
* Changing system settings
* Editing protected files

Windows requires a UAC confirmation before using this token.

---

# UAC Example

Opening Command Prompt normally:

```
Medium Integrity
```

Running as Administrator:

```
High Integrity
```

---

# Summary

Windows permissions are built around several layers:

```
Authentication
        ↓
Session Creation
        ↓
Access Token Generation
        ↓
Authorization Check
        ↓
ACL Permission Check
        ↓
Integrity Level Check
        ↓
Access Granted / Denied
```

Important concepts to remember:

* **Authentication** verifies identity.
* **Authorization** decides what users can do.
* **SIDs** uniquely identify users and groups.
* **Access Tokens** store security information during sessions.
* **ACLs and ACEs** control file permissions.
* **MIC** prevents lower-integrity processes from accessing higher-level resources.
* **UAC** limits administrator privileges by default.

Understanding these concepts is essential for Windows administration, Active Directory, incident response, and Windows privilege escalation.
