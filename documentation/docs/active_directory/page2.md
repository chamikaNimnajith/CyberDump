# Active Directory Network Enumeration

When assessing an Active Directory environment, one of the first steps is to understand the **network attack surface**.

The attack surface consists of the network services, ports, applications, and protocols that are exposed by a system.

For example, a Domain Controller may expose services such as:

```text
DNS       → 53
Kerberos  → 88
LDAP      → 389
SMB       → 445
LDAPS     → 636
WinRM     → 5985
```

By identifying these services, a security professional can understand what the target is running and determine which areas require further investigation.

---

# 1. Nmap Scanning Strategy

**Nmap** is one of the most commonly used tools for network discovery and port scanning.

A typical scan against a Domain Controller may look like:

```bash
nmap -Pn -sV -sC <target-ip>
```

Each option has a specific purpose.

---

## `-Pn` — Skip Host Discovery

Normally, Nmap first attempts to determine whether a host is online.

It may use techniques such as ICMP echo requests (ping) or other host-discovery probes.

However, Windows systems may block ICMP traffic.

As a result, Nmap might incorrectly conclude:

```text
Host seems down
```

The `-Pn` option tells Nmap:

> **Assume the host is online and scan it anyway.**

Example:

```bash
nmap -Pn <target-ip>
```

This is particularly useful when scanning systems that do not respond to traditional ping requests.

---

## `-sV` — Service Version Detection

The `-sV` option attempts to determine which service and software version are running on an open port.

For example:

```text
80/tcp   open   http    Microsoft IIS httpd 10.0
445/tcp  open   microsoft-ds
5985/tcp open   wsman
```

Instead of simply knowing:

```text
Port 80 is open
```

we may learn:

```text
Port 80 is running Microsoft IIS 10.0
```

This information helps with:

* Service identification
* Vulnerability research
* Technology fingerprinting
* Further enumeration

---

## `-sC` — Default NSE Scripts

The `-sC` option runs Nmap's **default NSE (Nmap Scripting Engine) scripts**.

These scripts perform common checks and gather additional information about discovered services.

For example, scripts may help identify:

* SMB information
* HTTP page titles
* SSL/TLS configuration
* Common service information
* Anonymous access
* Host and network details

A common enumeration command is therefore:

```bash
nmap -Pn -sV -sC <target-ip>
```

### Understanding the Scan

The three options can be remembered as:

| Option | Purpose                            |
| ------ | ---------------------------------- |
| `-Pn`  | Don't rely on ping/host discovery  |
| `-sV`  | Identify service versions          |
| `-sC`  | Run common NSE enumeration scripts |

---

# 2. Network Architecture and Pivoting

Before scanning a Domain Controller, it is important to understand **how the attacker can reach it**.

In a simple lab environment, the attacker and Domain Controller may be connected to the same network:

```text
┌────────────────┐
│ Attacker       │
│ 192.168.1.10   │
└───────┬────────┘
        │
        │ Same Network
        │
┌───────▼────────┐
│ Domain         │
│ Controller     │
│ 192.168.1.20   │
└────────────────┘
```

The attacker can directly scan the Domain Controller.

---

## Real-World Network Architecture

Production networks are usually more segmented.

A Domain Controller is generally placed inside an internal network rather than being directly exposed to the public Internet.

For example:

```text
Internet
    │
    ↓
Public Web Server
    │
    ↓
Internal Network
    │
    ↓
Domain Controller
```

An attacker may therefore need to compromise an initial system before being able to communicate with internal systems.

For example:

```text
Attacker
   │
   ↓
Internet-Facing Server
   │
   ↓
Compromised Host
   │
   ↓
Pivot / Tunnel
   │
   ↓
Internal Network
   │
   ↓
Domain Controller
```

This technique is known as **pivoting**.

---

## What Is Pivoting?

Pivoting means using a compromised system as a gateway to reach systems that were not directly accessible from the attacker's original machine.

For example:

```text
Attacker
   │
   │ Direct connection
   ↓
Compromised Machine
   │
   │ Internal connection
   ↓
Domain Controller
```

Tools such as SSH can be used to create tunnels or port forwarding in authorized testing environments.

> **Important:** Pivoting should only be performed against systems where you have explicit authorization.

---

# 3. Active Directory Ports and Services

Domain Controllers typically expose several network services.

Understanding these ports is extremely useful when performing Active Directory enumeration.

---

# 3.1 Port 53 — DNS

**DNS (Domain Name System)** translates human-readable names into IP addresses.

For example:

```text
dc01.example.local
        ↓
192.168.1.20
```

Active Directory heavily depends on DNS.

Clients use DNS to locate important domain services, including Domain Controllers and Kerberos services.

### Why DNS Is Important in AD

When a computer needs to communicate with a Domain Controller, it needs to know:

```text
Where is the Domain Controller?
Where is the Kerberos service?
Where are other domain services?
```

DNS provides this information.

### Useful Tool

Windows provides:

```cmd
nslookup dc01.example.local
```

This can be used to resolve a hostname to an IP address.

### Security Consideration

Misconfigured DNS servers may expose unnecessary information.

For example, a **DNS zone transfer** can potentially reveal large amounts of DNS information if the server is incorrectly configured.

---

# 3.2 Port 80 — HTTP

Port **80** is commonly used for HTTP.

HTTP provides unencrypted web communication.

For example:

```bash
curl -v http://<target-ip>
```

A server may return information such as:

```text
Server: Microsoft-IIS/10.0
```

This tells us that the target is running **Microsoft IIS**.

### Security Considerations

HTTP does not provide encryption.

Sensitive information sent over HTTP can potentially be intercepted or modified by an attacker positioned on the network.

Running unnecessary web applications on a Domain Controller also increases its attack surface.

> **Security principle:** A Domain Controller should run only the services that are actually required.

---

# 3.3 Port 88 — Kerberos

Port **88** is used by **Kerberos**, the primary authentication protocol used by Active Directory.

Kerberos allows users and computers to authenticate to network services using cryptographic tickets.

The main Kerberos components include:

* **Authentication Service (AS)**
* **Ticket Granting Service (TGS)**
* **Key Distribution Center (KDC)**

A simplified process looks like:

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

### Security Consideration

Kerberos is an important area of Active Directory security.

For example, **Kerberoasting** is an attack technique that abuses service account Kerberos tickets to attempt offline password cracking.

---

# 3.4 Port 135 — Microsoft RPC

Port **135** is used by Microsoft's **Remote Procedure Call (RPC)** infrastructure.

The service running on this port is known as the **RPC Endpoint Mapper**.

Its job is essentially to help clients discover which dynamic port a particular RPC service is using.

Think of it as a directory:

```text
Client
  │
  │ "Where is RPC service X?"
  ↓
Port 135
  │
  ↓
Endpoint Mapper
  │
  ↓
Dynamic RPC Port
```

RPC is used by many Windows services and administrative functions.

### Enumeration

With appropriate authorized credentials, tools such as `rpcclient` can be used to query Windows systems.

For example:

```bash
rpcclient <target-ip> -U <username>
```

Within an authenticated RPC session, commands such as:

```text
enumdomusers
```

can enumerate domain users when the account has the necessary access.

Potential accounts may include:

```text
Administrator
Guest
krbtgt
```

---

# 3.5 Port 139 — NetBIOS

Port **139** is associated with **NetBIOS Session Service**.

NetBIOS is an older Windows networking technology that was historically used for:

* File sharing
* Printer sharing
* Computer name discovery

Modern Windows networks primarily use SMB directly over port **445**, so port 139 is considered a legacy service.

Tools such as `nbtscan` can help retrieve information such as:

* NetBIOS names
* Workgroup/domain names
* Host information

---

# 3.6 Port 389 — LDAP

Port **389** is the standard port for **LDAP (Lightweight Directory Access Protocol)**.

LDAP provides a way to query and interact with directory services.

In Active Directory, LDAP can be used to retrieve information about:

* Users
* Groups
* Computers
* Organizational Units
* Other directory objects

A simplified view is:

```text
LDAP Client
     │
     ↓
LDAP
     │
     ↓
Active Directory
     │
     ├── Users
     ├── Groups
     ├── Computers
     └── OUs
```

### LDAP Enumeration

On Linux, `ldapsearch` can be used to query LDAP servers.

A client normally performs a **bind**, which establishes an LDAP session and identifies the credentials or authentication context being used.

On Windows, PowerShell and .NET directory APIs can also be used to query Active Directory.

---

# 3.7 Port 443 — HTTPS

Port **443** is normally used for **HTTPS**.

HTTPS is HTTP protected by **TLS (Transport Layer Security)**.

It provides:

* Encryption
* Data integrity
* Server authentication

The basic difference is:

```text
HTTP
Port 80
   ↓
Unencrypted

HTTPS
Port 443
   ↓
TLS encrypted
```

HTTPS can be used by many different applications and is not exclusive to Active Directory.

In environments using **Active Directory Certificate Services (AD CS)**, certificates issued by an organization's Certificate Authority may also be used to support secure services and authentication mechanisms.

---

# 3.8 Port 445 — SMB

Port **445** is used by **SMB (Server Message Block)**.

SMB is one of the most important Windows network protocols.

It is commonly used for:

* File sharing
* Printer sharing
* Windows administrative operations
* Inter-process communication
* Remote resource access

For example:

```text
Windows Server
     │
     └── C:\CompanyShare
             │
             ↓
      \\DC01\CompanyShare
```

A client can access the shared resource over SMB.

---

## SMB Versions

Older environments may use **SMBv1**.

SMBv1 is obsolete and should be disabled because of serious security weaknesses.

Modern Windows environments use:

* SMBv2
* SMBv3

SMBv3 also supports security features such as encryption.

---

## SMB Shares and Security

Incorrectly configured SMB shares can expose sensitive information.

For example:

```text
\\DC01\Public
      │
      ├── Documents
      ├── Backups
      └── Configuration Files
```

Security professionals can use authorized enumeration tools such as:

```text
smbclient
smbmap
```

to understand available shares and their permissions.

The key question is:

> **Who can access this share, and what can they do with it?**

---

# 3.9 Port 464 — Kerberos Password Change

Port **464** is associated with Kerberos password-change operations.

It allows Kerberos clients to securely change passwords in environments using Kerberos-based authentication.

It works alongside the main Kerberos services on port 88.

Remember:

```text
88  → Kerberos authentication
464 → Kerberos password changes
```

---

# 3.10 Port 593 — RPC over HTTP

Port **593** is associated with **RPC over HTTP**.

It allows certain RPC communications to be transported through HTTP.

This can support Windows technologies such as **Distributed Component Object Model (DCOM)**.

It is another example of how Windows networking uses multiple protocols to provide remote management and communication functionality.

---

# 3.11 Port 636 — LDAPS

Port **636** is commonly used for **LDAPS (LDAP over TLS)**.

It provides encrypted LDAP communication.

The basic difference is:

```text
389
 ↓
LDAP
 ↓
Standard LDAP communication

636
 ↓
LDAPS
 ↓
LDAP protected by TLS
```

LDAPS helps protect directory queries and authentication information while it travels across the network.

---

# 3.12 Ports 3268 and 3269 — Global Catalog

The **Global Catalog (GC)** provides directory information across the entire Active Directory forest.

Two commonly associated ports are:

| Port     | Service                      |
| -------- | ---------------------------- |
| **3268** | Global Catalog over LDAP     |
| **3269** | Global Catalog over LDAP/TLS |

The Global Catalog contains a partial, read-only copy of objects from domains across the forest.

This allows applications and users to perform forest-wide searches efficiently.

---

# 3.13 Port 5357 — Microsoft HTTP API

Port **5357** is associated with Microsoft's HTTP-based services, particularly **Web Services for Devices (WSD)**.

These services can help Windows discover network devices such as:

* Printers
* Scanners
* Other compatible devices

It is not one of the core Active Directory authentication services, but it may appear during a Windows network scan.

---

# 3.14 Port 5985 — WinRM

Port **5985** is commonly used by **Windows Remote Management (WinRM)** over HTTP.

WinRM provides a mechanism for remotely managing Windows systems.

It can be used by administrators for tasks such as:

* Remote PowerShell
* System administration
* Remote management

Windows administrators can check whether the WinRM service is running with:

```powershell
Get-Service WinRM
```

A related secure WinRM configuration commonly uses port **5986** with HTTPS.

---

# 4. Other Important Network Services

Not every useful service will necessarily appear on a particular Domain Controller scan.

Security professionals should also understand several common network services.

---

## Port 21 — FTP

**FTP (File Transfer Protocol)** is used to transfer files between systems.

```text
21 → FTP
```

FTP is an old protocol and does not provide encryption by itself.

Therefore, sensitive credentials and files should not normally be transferred using plain FTP.

---

## Port 22 — SSH

**SSH (Secure Shell)** provides encrypted remote administration.

```text
22 → SSH
```

SSH is commonly used for:

* Remote terminal access
* Secure file transfers
* Server administration
* Port forwarding
* Tunneling

In authorized penetration tests, SSH port forwarding can also be used to create a controlled communication path into an internal network.

For example:

```text
Attacker
   │
   │ SSH tunnel
   ↓
Compromised Server
   │
   ↓
Internal Network
```

---

# 5. Port 1433 — Microsoft SQL Server

Port **1433** is the default TCP port commonly associated with **Microsoft SQL Server (MSSQL)**.

SQL Server is used to store and manage relational databases.

For example:

```text
Application
    │
    ↓
MSSQL Server
    │
    ↓
Database
```

Security professionals may use database clients to interact with an authorized SQL Server instance.

Tools such as Impacket's `mssqlclient.py` can support SQL Server connections in appropriate testing environments.

Poorly secured database configurations can create serious security risks, including excessive privileges and, in some circumstances, paths to operating-system command execution.

---

# 6. Port 3389 — RDP

**RDP (Remote Desktop Protocol)** is Microsoft's protocol for remotely accessing a Windows graphical desktop.

```text
Administrator
      │
      │ RDP
      ↓
Windows Server
      │
      ↓
Remote Desktop
```

RDP allows an administrator to interact with the remote Windows desktop as if they were sitting in front of the machine.

It is commonly used for:

* Server administration
* Remote troubleshooting
* Desktop access

Because RDP provides powerful remote access, it should be properly protected using strong authentication, network restrictions, and other security controls.

---

# 7. Active Directory Port Cheat Sheet

The following table is useful as a quick reference when enumerating a Domain Controller.

|     Port | Protocol / Service      | Main Purpose                               |
| -------: | ----------------------- | ------------------------------------------ |
|   **21** | FTP                     | File transfer                              |
|   **22** | SSH                     | Secure remote administration and tunneling |
|   **53** | DNS                     | Name resolution and AD service discovery   |
|   **80** | HTTP                    | Web services                               |
|   **88** | Kerberos                | AD authentication                          |
|  **135** | MS RPC                  | RPC Endpoint Mapper                        |
|  **139** | NetBIOS                 | Legacy Windows networking                  |
|  **389** | LDAP                    | Active Directory directory queries         |
|  **443** | HTTPS                   | TLS-protected web services                 |
|  **445** | SMB                     | Windows file/resource sharing              |
|  **464** | Kerberos                | Password changes                           |
|  **593** | RPC over HTTP           | RPC communications                         |
|  **636** | LDAPS                   | LDAP over TLS                              |
| **1433** | MSSQL                   | Microsoft SQL Server                       |
| **3268** | Global Catalog          | Forest-wide LDAP queries                   |
| **3269** | Global Catalog over TLS | Secure forest-wide queries                 |
| **3389** | RDP                     | Remote Windows desktop                     |
| **5357** | Microsoft HTTP API      | Web Services for Devices                   |
| **5985** | WinRM                   | Windows remote management                  |

---

# 8. How to Think About a Domain Controller Scan

When a scan returns many ports, don't simply memorize the numbers.

Instead, ask:

> **What does this service tell me about the environment?**

For example:

```text
53  → DNS
      ↓
      Helps identify the domain and AD infrastructure

88  → Kerberos
      ↓
      Confirms AD authentication infrastructure

389 → LDAP
      ↓
      Directory information may be available

445 → SMB
      ↓
      Windows shares and resources may exist

3268 → Global Catalog
       ↓
       Forest-wide directory information

5985 → WinRM
       ↓
       Remote Windows management may be available
```

This approach makes enumeration much more useful than simply memorizing port numbers.

---

# 9. A Typical AD Enumeration Workflow

A simplified workflow might look like:

```text
              Start
                │
                ↓
        Identify target host
                │
                ↓
        Scan TCP ports
                │
                ↓
       Identify services
                │
                ↓
       Identify AD services
                │
        ┌───────┼────────┐
        ↓       ↓        ↓
       DNS    LDAP      SMB
        │       │        │
        ↓       ↓        ↓
      Domain   Users    Shares
      info     /Groups  /Resources
        │       │        │
        └───────┼────────┘
                ↓
        Build AD attack-surface
             understanding
```

The goal of initial enumeration is not to immediately exploit everything.

The goal is to **build an accurate picture of the environment**.

---

# 10. Key Takeaways

When working with Active Directory environments, remember these important points:

* **Nmap** helps identify open ports and services.
* `-Pn` tells Nmap to scan even when host discovery fails.
* `-sV` identifies service versions.
* `-sC` runs useful default NSE scripts.
* **DNS (53)** is fundamental to Active Directory.
* **Kerberos (88)** handles the primary AD authentication process.
* **LDAP (389)** provides directory access.
* **SMB (445)** is central to Windows file and resource sharing.
* **LDAPS (636)** provides LDAP over TLS.
* **Global Catalog (3268/3269)** provides forest-wide directory searches.
* **WinRM (5985)** enables Windows remote management.
* **RDP (3389)** provides graphical remote access.
* **RPC (135)** supports many Windows management and communication functions.
* **Pivoting** can provide access to internal systems when they are not directly reachable.
* A port number alone is not enough—always understand **what service is running and why it is exposed**.

> **The main goal of network enumeration is to transform an unknown system into an understandable map of services, protocols, relationships, and potential security boundaries.**
