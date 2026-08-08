# SMB Enumeration and Common SMB Attacks

**SMB (Server Message Block)** is one of the most important protocols in Windows networks.

It is commonly used for:

* File and folder sharing
* Printer sharing
* Remote administration
* Inter-process communication
* Accessing Windows administrative shares

Because SMB is deeply integrated into Windows and Active Directory, it is an important area to investigate during an authorized security assessment.

This section introduces several tools used for SMB enumeration and explains common SMB-related security threats.

> **Important:** The techniques described below should only be performed against systems you own or have explicit permission to test.

---

# 1. Important SMB Security Tools

Several tools are commonly used when investigating Windows and SMB environments.

---

## 1.1 SMBMap

**SMBMap** is a Python-based tool designed to make SMB enumeration easier.

It can help identify:

* Available SMB shares
* Share permissions
* Accessible files
* Operating system information
* Whether the current credentials have administrative access

A simple way to think about SMBMap is:

> **SMBMap helps you quickly understand what SMB resources a user can access.**

---

## 1.2 smbclient

**smbclient** is a command-line SMB client commonly available on Linux systems.

It can be used to:

* List SMB shares
* Connect to a share
* Browse directories
* Download files
* Upload files

For example:

```bash
smbclient -L //<target-ip> -U <username>
```

The `-L` option requests a list of available shares.

After connecting to a share, the interface looks somewhat like an FTP session:

```text
smb: \> ls
smb: \> cd Documents
smb: \> get report.txt
```

This makes `smbclient` particularly useful for manually investigating SMB shares.

---

# 1.3 NetExec (`nxc`)

**NetExec**, commonly executed using:

```bash
nxc
```

is a powerful network authentication and enumeration framework.

It supports multiple protocols, including:

* SMB
* WinRM
* LDAP
* MSSQL
* WMI
* SSH
* VNC

It can be used to automate many administrative and security-testing tasks across multiple Windows systems.

For example, security professionals can use it to:

* Enumerate SMB hosts
* Test authorized credentials
* Identify administrative access
* Enumerate users and shares
* Perform controlled password auditing
* Execute commands when appropriate privileges exist

> NetExec is the modern successor to the tool formerly known as CrackMapExec.

---

# 1.4 Legba

**Legba** is a high-performance, multi-protocol authentication auditing tool.

It can be used to test large credential lists against supported services.

For example:

```text
Username list
      +
Password list
      ↓
 Authentication testing
      ↓
Valid / Invalid credentials
```

It can be useful for authorized password auditing and security assessments.

---

# 1.5 Responder

**Responder** is a network security-testing tool that can emulate several network services.

One of its well-known uses is setting up fake services that cause Windows clients to attempt authentication.

For example:

```text
Windows Client
      │
      │ Authentication attempt
      ↓
Fake SMB Server
      │
      ↓
NetNTLM challenge-response
```

The resulting authentication material can potentially be captured and analyzed.

---

# 1.6 ntlmrelayx

**ntlmrelayx** is part of the **Impacket** toolkit.

It is commonly associated with **NTLM relay attacks**.

Instead of attempting to crack captured authentication data, a relay attack attempts to forward the authentication exchange to another service in real time.

Conceptually:

```text
Victim
  │
  │ NTLM authentication
  ↓
Attacker
  │
  │ Relay
  ↓
Target Server
```

If the target accepts the relayed authentication and the account has sufficient privileges, the attack may result in unauthorized access.

---

# 1.7 Proxychains

**Proxychains** allows applications to route their network connections through configured proxy servers.

It is particularly useful when a security tester has established an authorized SOCKS proxy into an internal network.

For example:

```text
Security Tester
      │
      ↓
SOCKS Proxy
      │
      ↓
Internal Network
      │
      ↓
Target
```

This can allow tools that normally cannot reach an internal host to communicate through the proxy.

---

# 2. SMB Threats

There are several important security problems associated with SMB.

They range from simple configuration mistakes to serious authentication attacks.

The major topics covered here are:

1. Guest/anonymous access
2. Administrative share abuse
3. Password brute forcing
4. Password spraying
5. SMBv1 vulnerabilities
6. NetNTLM capture
7. Pass-the-Hash
8. SMB relay

---

# 3. Guest and Anonymous SMB Access

Normally, users should authenticate before accessing protected SMB resources.

However, incorrectly configured systems may allow **anonymous or guest access**.

This means that a user may be able to enumerate or access SMB resources without valid domain credentials.

---

## What Is a Null Session?

A **null session** refers to an SMB connection where no username or password is provided.

Conceptually:

```text
Username: ""
Password: ""
```

If the server incorrectly permits this type of connection, information about the SMB environment may become accessible.

For example, an attacker may discover:

* Share names
* Domain information
* User information
* Other directory information

---

## Guest Access

Guest access is slightly different.

A client may authenticate using the built-in **Guest** account or be mapped to a guest session.

This can happen when a Windows environment has been configured to allow anonymous users to access resources through the Guest account.

The important security lesson is:

> **SMB should not allow unauthenticated users to access sensitive resources.**

---

## Why Is It Dangerous?

Suppose a Domain Controller exposes:

```text
\\DC01\Public
\\DC01\Backup
\\DC01\Users
```

If anonymous users can access these shares, they may be able to retrieve sensitive information.

That information could then help with further security testing.

---

# 4. Remote Code Execution Through Administrative Shares

Windows provides several hidden administrative shares.

Common examples include:

```text
C$
ADMIN$
IPC$
```

For example:

```text
C$
 ↓
Provides administrative access to the C: drive
```

These shares are intended for administrative purposes and are normally protected by Windows authentication and authorization.

---

## Why Are They Important?

Suppose a user has administrative privileges on a Windows machine.

That user may be able to access:

```text
\\TARGET\C$
```

and interact with the remote filesystem.

Administrative privileges may also allow remote management mechanisms to execute commands.

Conceptually:

```text
Administrator Credentials
          ↓
Administrative SMB Access
          ↓
Remote System Management
          ↓
Potential Command Execution
```

Tools such as NetExec can identify whether supplied credentials provide administrative access.

For example, an output may indicate administrative privileges with a status such as:

```text
Pwn3d!
```

This should be interpreted as:

> **The supplied account has administrative-level access to the target through the tested protocol.**

It does not mean the system is automatically compromised in every possible way.

---

# 5. Password Brute Forcing

**Brute forcing** means repeatedly attempting different passwords against a particular account.

For example:

```text
Username: administrator

Password attempts:
123456
password
Password123
Welcome1
Winter2026
...
```

The attacker keeps testing passwords until:

* A valid password is found
* The wordlist is exhausted
* The account becomes locked
* Defensive controls stop the attempts

---

## Why Brute Force Is Dangerous

Repeated login attempts can:

* Trigger account lockouts
* Generate security alerts
* Increase network traffic
* Potentially compromise weak accounts

Organizations should therefore use:

* Strong passwords
* Account lockout policies
* Multi-factor authentication where appropriate
* Rate limiting
* Monitoring and alerting

---

## Guest Account Complication

If SMB Guest access is enabled, authentication testing can produce misleading results.

For example, multiple incorrect username/password combinations may appear successful because the server maps unauthenticated requests to the Guest account.

Therefore:

> **A successful SMB connection does not always mean the supplied username and password were actually valid.**

This is an important point when interpreting automated tool output.

---

# 6. Password Spraying

Password spraying is different from brute forcing.

### Brute Force

One account:

```text
Administrator
    ↓
Password1
Password2
Password3
Password4
...
```

### Password Spraying

Many accounts:

```text
User1 ── Password123
User2 ── Password123
User3 ── Password123
User4 ── Password123
```

The goal is to test a **small number of commonly used passwords across many accounts**.

---

## Why Use Password Spraying?

Traditional brute forcing may cause an account to become locked.

For example:

```text
5 failed attempts
      ↓
Account locked
```

Password spraying reduces the number of attempts against each individual account.

For example:

```text
Password123
     ↓
100 accounts
     ↓
1 attempt per account
```

However, password spraying can still trigger security controls and may violate organizational policies.

---

## Defensive Controls

Organizations can reduce the risk using:

* Strong password policies
* MFA
* Smart lockout policies
* Monitoring authentication failures
* Detection of unusual login patterns
* Blocking commonly used passwords

---

# 7. SMBv1 and EternalBlue

Older versions of SMB can introduce serious security risks.

The most famous example is **SMBv1** and the **EternalBlue** vulnerability.

---

## What Is EternalBlue?

**EternalBlue** refers to an exploit associated with:

```text
MS17-010
CVE-2017-0144
```

The vulnerability affected Microsoft's implementation of SMBv1.

Under vulnerable conditions, an unauthenticated attacker could potentially achieve **remote code execution**.

The vulnerability became particularly well known because it was used by malware including the **WannaCry ransomware**.

---

## Why SMBv1 Is Dangerous

SMBv1 is old and contains significant security weaknesses.

Modern Windows environments should generally use newer SMB versions such as:

```text
SMBv2
SMBv3
```

Organizations should disable SMBv1 wherever it is no longer required.

---

## Vulnerability Assessment

During an authorized security assessment, tools such as Nmap can be used to identify potentially vulnerable systems.

For example, Nmap's SMB vulnerability checks can identify systems affected by known SMB vulnerabilities.

The important concept is:

```text
Discover hosts
      ↓
Identify SMB
      ↓
Determine SMB version
      ↓
Check for known vulnerabilities
      ↓
Remediate vulnerable systems
```

> Never deploy an exploit against systems without explicit authorization.

---

# 8. NetNTLM Hash Capture

One of the most important concepts in Windows authentication security is the difference between a **password** and an **NTLM challenge-response exchange**.

When NTLM authentication is used, Windows does not simply send the user's plaintext password over the network.

Instead, the authentication process involves a challenge-response mechanism.

A simplified flow is:

```text
Client                    Server
  │                         │
  │──── Authentication ────→│
  │                         │
  │←──── Challenge ─────────│
  │                         │
  │──── Response ──────────→│
  │                         │
  │       Authentication    │
```

The response can contain information derived from the user's password.

This is commonly referred to as **NetNTLM** authentication material.

---

## NetNTLMv1 vs NetNTLMv2

Two commonly encountered versions are:

```text
NetNTLMv1
NetNTLMv2
```

NetNTLMv2 is generally stronger than NetNTLMv1, but capturing authentication material can still create security risks.

---

# 9. Capturing NetNTLM Authentication

Tools such as Responder can emulate network services and potentially cause Windows clients to authenticate to the attacker's system.

Conceptually:

```text
Victim Computer
      │
      │ Attempts authentication
      ↓
Fake Network Service
      │
      ↓
NetNTLM Authentication Material
```

The captured material can potentially be subjected to **offline password cracking**.

Tools commonly used for authorized password auditing include:

* Hashcat
* John the Ripper

The key point is:

> **A captured NetNTLM challenge-response is not the same thing as the user's NTLM password hash.**

---

# 10. NetNTLM vs NTLM Hash

This distinction is extremely important.

| Type          | Description                                                          | Pass-the-Hash?                   |
| ------------- | -------------------------------------------------------------------- | -------------------------------- |
| **NTLM hash** | Static hash derived from the user's password                         | Yes, under applicable conditions |
| **NetNTLM**   | Challenge-response authentication exchange captured from the network | No                               |

For example:

```text
Captured through network authentication
        ↓
     NetNTLM
        ↓
Usually requires offline cracking
```

Whereas:

```text
Obtained from credential storage
        ↓
     NTLM hash
        ↓
Can potentially be used for Pass-the-Hash
```

---

# 11. Pass-the-Hash (PtH)

**Pass-the-Hash** is an authentication technique where an attacker uses a user's **NTLM hash** instead of knowing the user's plaintext password.

Normally:

```text
Username
   +
Password
   ↓
Authentication
```

With Pass-the-Hash:

```text
Username
   +
NTLM Hash
   ↓
Authentication
```

The password itself does not need to be recovered.

---

## Where Can NTLM Hashes Come From?

In an authorized security assessment, NTLM hashes may potentially be exposed through compromised credential stores or memory.

For example, tools such as **Mimikatz** are commonly associated with credential extraction from Windows systems.

The security lesson is:

> **Protecting password hashes is important because an attacker may not need to recover the original password to abuse them.**

---

## Pass-the-Hash vs NetNTLM

Remember:

```text
NetNTLM capture
      ↓
Challenge-response
      ↓
Cannot directly perform PtH

NTLM hash
      ↓
Static credential hash
      ↓
Can potentially perform PtH
```

This distinction is one of the most important concepts when learning Windows authentication attacks.

---

# 12. SMB Relay Attacks

An **SMB relay attack** is different from Pass-the-Hash.

Instead of stealing a static NTLM hash and using it later, the attacker attempts to **relay a live NTLM authentication exchange to another server**.

The basic idea is:

```text
Victim
  │
  │ NTLM authentication
  ↓
Attacker
  │
  │ Relay authentication
  ↓
Target Server
```

The attacker does not necessarily need to know the victim's password.

---

# 13. Why SMB Signing Matters

**SMB signing** adds cryptographic protection to SMB communications.

It helps ensure that SMB messages have not been modified or improperly relayed.

SMB relay attacks become significantly harder when SMB signing is properly required.

Therefore, one important security assessment question is:

> **Is SMB signing required on the target?**

---

## Secure Configuration

Ideally:

```text
SMB
 │
 └── Signing required
          ↓
     Better protection
```

Potentially weaker configuration:

```text
SMB
 │
 └── Signing not required
          ↓
     Relay attacks may be possible
```

The exact exploitability of an environment depends on additional configuration and authentication conditions.

---

# 14. Conceptual SMB Relay Flow

A simplified relay attack looks like this:

```text
                 Authentication
Victim ─────────────────────────→ Attacker
                                      │
                                      │ Relay
                                      ↓
                                  Target Server
                                      │
                                      ↓
                              Authentication result
```

If the target accepts the authentication and the account has sufficient privileges, the attacker may gain access to resources available to that account.

---

# 15. SOCKS Proxies and Internal Access

Some security-testing tools can expose authenticated sessions through a **SOCKS proxy**.

A SOCKS proxy acts as an intermediary between the security tester and another system.

For example:

```text
Security Tester
      │
      ↓
SOCKS Proxy
      │
      ↓
Authenticated Session
      │
      ↓
Internal Target
```

Tools such as Proxychains can then route compatible applications through the SOCKS proxy.

Conceptually:

```text
proxychains <tool>
       │
       ↓
SOCKS Proxy
       │
       ↓
Internal Target
```

This is particularly useful in authorized assessments where a tester needs to interact with internal systems through an established network path.

---

# 16. Comparing the Major SMB Attacks

It is useful to understand how the attacks differ.

| Attack                         | Main Idea                                                        |
| ------------------------------ | ---------------------------------------------------------------- |
| **Anonymous/Guest Access**     | Access SMB without proper credentials                            |
| **Administrative Share Abuse** | Use legitimate admin credentials to access administrative shares |
| **Brute Force**                | Try many passwords against one account                           |
| **Password Spraying**          | Try a small number of passwords against many accounts            |
| **EternalBlue**                | Exploit a vulnerable SMBv1 implementation                        |
| **NetNTLM Capture**            | Capture a challenge-response authentication exchange             |
| **Pass-the-Hash**              | Authenticate using a stolen NTLM hash                            |
| **SMB Relay**                  | Forward a live NTLM authentication exchange to another target    |

---

# 17. The Most Important Concepts to Remember

### 1. SMB is a major Windows attack surface

SMB is deeply integrated into Windows, so exposed SMB services deserve careful security review.

### 2. Authentication does not always mean authorization

A successful login does not automatically mean the user has administrative privileges.

```text
Authentication
     ↓
Who are you?

Authorization
     ↓
What are you allowed to do?
```

### 3. NTLM and NetNTLM are different

This is critical:

```text
NTLM Hash
   ↓
Potentially usable for Pass-the-Hash

NetNTLM
   ↓
Captured challenge-response
   ↓
Cannot directly be used for Pass-the-Hash
```

### 4. Brute force and password spraying are different

```text
Brute Force
1 user → many passwords

Password Spraying
Many users → few passwords
```

### 5. SMB signing is an important defense

Requiring SMB signing helps protect against certain relay attacks.

### 6. Disable legacy protocols

SMBv1 should generally be disabled because of its security weaknesses and historical vulnerabilities such as EternalBlue.

### 7. Least privilege matters

Administrative SMB shares and remote management become particularly dangerous when excessive privileges are assigned to ordinary accounts.

---

# 18. Defensive Checklist

Organizations can reduce SMB-related risks by applying the following controls:

* Disable **SMBv1**.
* Require **SMB signing** where appropriate.
* Disable unnecessary **Guest and anonymous access**.
* Restrict access to administrative shares.
* Use strong, unique passwords.
* Deploy **MFA** where supported.
* Implement appropriate account lockout and smart-lockout policies.
* Monitor failed authentication attempts.
* Detect unusual SMB authentication activity.
* Restrict unnecessary SMB access between network segments.
* Keep Windows systems patched.
* Minimize unnecessary services on Domain Controllers.
* Follow the **principle of least privilege**.
* Protect credential material such as NTLM hashes.
* Segment Domain Controllers from untrusted networks.
* Regularly audit SMB shares and their permissions.

---

# 19. Final Mental Model

A useful way to understand SMB security is to think about the different stages an attacker might encounter:

```text
                    SMB
                     │
        ┌────────────┼─────────────┐
        ↓            ↓             ↓
    Enumeration   Authentication   Vulnerabilities
        │            │             │
        ↓            ↓             ↓
     Shares       Credentials    SMBv1
     Users        NTLM           Known CVEs
     Hosts        Kerberos
        │            │
        └──────┬─────┘
               ↓
          Authorization
               │
       ┌───────┴────────┐
       ↓                ↓
   Normal Access    Excessive Access
                         │
                         ↓
                   Security Impact
```

The key lesson is that SMB security is not just about finding open port **445**.

A proper assessment should consider:

> **What SMB services are exposed? Who can authenticate? What resources can they access? Are legacy protocols enabled? Are credentials protected? Is SMB signing required? And are users receiving more privileges than they actually need?**

Understanding these questions provides a strong foundation for both **Active Directory penetration testing** and **Windows network defense**.
