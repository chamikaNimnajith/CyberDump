# System Hardening, Maintenance, and Virtualization Security

A secure system does not become secure simply because a firewall or antivirus program has been installed. Security must be considered throughout the **entire system lifecycle**, beginning with planning and continuing through deployment, maintenance, monitoring, and eventual recovery.

System security therefore involves several connected activities:

```text
Security Planning
       ↓
OS Hardening
       ↓
Application Security
       ↓
Network & Host Protection
       ↓
Continuous Monitoring
       ↓
Backup & Recovery
       ↓
Security Improvement
```

This page explores these principles and their application to operating systems, applications, virtual machines, software-defined networks, and containers.

---

# 1. Fundamental Security Mitigation Strategies

One of the most effective ways to improve security is to implement a small number of high-impact controls correctly.

The **Australian Signals Directorate (ASD)** published a well-known set of mitigation strategies known as the **Top 35 Mitigation Strategies**. The material emphasizes that more than 85% of targeted cyber intrusions could have been prevented by implementing the top four strategies.

These include:

1. Application allowlisting
2. Rapid application patching
3. Restricting administrative privileges
4. Implementing defense in depth

---

## 1.1 Application Allowlisting

**Application allowlisting** means allowing only approved applications to execute.

Instead of asking:

> "Which applications should be blocked?"

the organization asks:

> "Which applications are explicitly allowed to run?"

For example:

```text
Approved Applications
       │
       ├── Web Browser ✓
       ├── Office Suite ✓
       ├── Company App ✓
       └── Unknown Program ✗
```

This can prevent unauthorized or malicious programs from executing even if they reach the system.

---

## 1.2 Rapid Patch Management

Software vulnerabilities are frequently discovered after systems have already been deployed.

Organizations should therefore:

1. Monitor for security updates.
2. Test patches.
3. Deploy them quickly.
4. Verify successful installation.

```text
Vulnerability Discovered
          ↓
Security Patch Released
          ↓
Test
          ↓
Deploy
          ↓
Verify
```

Delaying important security updates increases the period during which attackers can exploit known vulnerabilities.

---

## 1.3 Restricting Administrative Privileges

Users should receive only the privileges required to perform their tasks.

For example:

```text
Normal Employee
      ↓
Standard User
      ↓
Limited Permissions
```

rather than:

```text
Normal Employee
      ↓
Administrator
      ↓
Complete System Control
```

This is an application of the **principle of least privilege**.

If an attacker compromises a standard account, the damage is generally more limited than if the compromised account has unrestricted administrative privileges.

---

## 1.4 Defense in Depth

**Defense in depth** means using multiple independent security controls.

For example:

```text
Internet
   ↓
Firewall
   ↓
IDS/IPS
   ↓
Network Segmentation
   ↓
Endpoint Security
   ↓
Access Control
   ↓
Encryption
   ↓
Backups
```

If one security control fails, another control may still prevent or limit the attack.

---

# 2. System Security Planning

Security should be considered **before a system is deployed**.

A new system should first undergo a security and risk assessment.

The organization needs to determine:

* What information will the system handle?
* Who will use it?
* What privileges will users require?
* How will users authenticate?
* Who will administer the system?
* What network services are required?
* What host-level security controls are necessary?
* What threats could affect the system?

The planning process can be represented as:

```text
Identify Requirements
        ↓
Risk Assessment
        ↓
Security Requirements
        ↓
Security Architecture
        ↓
Implementation
        ↓
Testing
        ↓
Deployment
```

Good planning attempts to achieve the best possible security while avoiding unnecessary costs.

---

# 3. Base Operating System Hardening

A newly installed operating system is rarely configured for maximum security.

Default configurations are generally designed to provide:

* Ease of installation
* Compatibility
* Convenience
* Broad functionality

Security hardening changes this configuration to reduce unnecessary exposure.

---

## 3.1 Hardening Process

A basic operating-system hardening process may include:

```text
Install OS
   ↓
Apply Security Updates
   ↓
Install Security Tools
   ↓
Disable Unnecessary Services
   ↓
Configure Permissions
   ↓
Configure Firewall
   ↓
Configure Monitoring
   ↓
Test
   ↓
Deploy
```

Security tools may include:

* Antivirus/endpoint protection
* Host-based firewalls
* Intrusion detection systems
* File integrity monitoring
* Logging tools

---

# 4. Secure System Installation

New systems should ideally be constructed in a **protected staging environment** rather than directly on an exposed production network.

For example:

```text
Protected Staging Network
          ↓
Install System
          ↓
Apply Updates
          ↓
Configure Security
          ↓
Test
          ↓
Production Network
```

This reduces the chance that an incompletely secured system will be compromised during installation.

---

# 5. Patch Testing and Deployment

Security patches should be validated before being deployed across production systems.

A common process is:

```text
Patch Released
     ↓
Test System
     ↓
Compatibility Testing
     ↓
Security Validation
     ↓
Production Deployment
```

This helps identify potential problems such as:

* Application incompatibility
* Service failures
* Configuration changes
* Performance issues

However, testing should not become an excuse for indefinitely delaying critical security patches.

---

# 6. Reducing the Attack Surface

The **attack surface** is the collection of points through which an attacker could potentially interact with or compromise a system.

For example:

```text
Open Ports
Running Services
User Accounts
Applications
Network Protocols
Remote Interfaces
```

The larger the attack surface, the more opportunities an attacker may have.

Therefore:

> **A secure system should expose only the functionality that is actually required.**

---

## 6.1 Removing Default Accounts

Default accounts can be targeted by attackers because their names and behaviors may be well known.

Organizations should:

* Remove unnecessary accounts
* Disable unused accounts
* Change default credentials
* Use strong authentication
* Regularly review accounts

---

## 6.2 Disabling Unnecessary Services

If a service is not required, it should generally be disabled.

For example:

```text
Required Service   → Enabled ✓
Unused Service     → Disabled ✓
```

Every unnecessary network service potentially adds another attack surface.

---

## 6.3 Disabling Unnecessary Protocols

Unused protocols should also be disabled where possible.

This reduces unnecessary communication paths and limits opportunities for exploitation.

---

## 6.4 Restricting Permissions

File and system permissions should follow the **least privilege principle**.

Users and applications should receive only the permissions necessary to perform their functions.

---

# 7. Application Security

Operating-system hardening alone is not enough. Applications must also be securely configured.

Important application security practices include:

* Changing default configurations
* Removing default accounts
* Disabling unnecessary scripts
* Restricting file permissions
* Protecting application data
* Limiting remote access
* Encrypting sensitive communication

---

## 7.1 Non-Default Data Paths

Applications sometimes use predictable default directories.

Changing default storage paths can reduce exposure to certain automated attacks, particularly when combined with proper access controls.

However, changing a path alone is not a security control. The underlying permissions must also be correctly configured.

---

## 7.2 Web and FTP Servers

Remote-access servers such as web and FTP servers should have carefully restricted permissions.

For example:

```text
Web Server
   ↓
Read access → Required web content
Write access → Only where absolutely necessary
```

A web server should not normally have unrestricted write access to sensitive system directories.

Excessive permissions can allow attackers to:

* Upload malicious files
* Modify web content
* Execute unauthorized programs
* Access sensitive information

---

# 8. Cryptography and Application Security

Encryption provides protection for information both **at rest** and **in transit**.

### Data at Rest

Information stored on:

* Hard drives
* SSDs
* Databases
* File systems
* Backup media

can be encrypted.

### Data in Transit

Information traveling across networks can be protected using protocols such as:

* TLS
* IPsec
* SSH

```text
Data
 ↓
Encryption
 ↓
Untrusted Network
 ↓
Decryption
 ↓
Authorized System
```

Cryptographic keys must be properly generated, stored, rotated, and protected.

Simply enabling encryption without managing keys securely does not provide complete protection.

---

# 9. Security as a Continuous Process

Security does not end when a system is deployed.

A system must be continuously:

* Monitored
* Patched
* Tested
* Backed up
* Audited
* Updated
* Recovered when incidents occur

This creates a continuous security lifecycle:

```text
Deploy
  ↓
Monitor
  ↓
Detect
  ↓
Respond
  ↓
Recover
  ↓
Improve
  ↺
```

This is particularly important because vulnerabilities and threats change over time.

---

# 10. Security Logging

**Logging** is a fundamental part of system security.

Logs can record events such as:

* User logins
* Failed authentication attempts
* Network connections
* Process execution
* File access
* Configuration changes
* Security alerts
* Administrative actions

For example:

```text
22:10:31 Login failure
22:10:35 Login failure
22:10:39 Login failure
22:10:44 Successful login
```

This sequence could indicate a possible brute-force attack.

---

## 10.1 Automated Log Analysis

Manually reviewing large numbers of logs is difficult.

Automated analysis can identify:

* Suspicious patterns
* Repeated failures
* Unusual login times
* Unexpected processes
* Network anomalies

Security information and event management systems can aggregate logs from multiple systems and help administrators reconstruct what happened during an incident.

Good logging therefore supports:

```text
Detection
   ↓
Investigation
   ↓
Incident Response
   ↓
Forensic Analysis
```

Logs should themselves be protected against unauthorized modification or deletion.

---

# 11. Backup and Archiving

**Backups** and **archives** are related but serve different purposes.

## Backup

A backup is a copy of data created primarily so that the original information can be restored after:

* Accidental deletion
* Hardware failure
* Malware
* Ransomware
* Data corruption
* Disaster

```text
Production Data
      ↓
   Backup
      ↓
Restore when necessary
```

Backups are generally created at regular intervals.

---

## Archive

An **archive** is intended primarily for long-term retention.

Organizations may retain archives because of:

* Business requirements
* Legal requirements
* Regulatory requirements
* Historical records
* Operational requirements

Therefore:

```text
Backup → Recovery
Archive → Long-Term Retention
```

The two should not be treated as identical.

---

# 12. Backup Storage Considerations

Backups can be classified according to where and how they are stored.

### Online Backups

Connected to the network and readily accessible.

**Advantages:**

* Fast recovery
* Easy automation

**Disadvantages:**

* Potentially vulnerable to ransomware
* Accessible to attackers if the backup environment is compromised

---

### Offline Backups

Disconnected from normal network access.

**Advantages:**

* Stronger protection against network-based attacks
* Ransomware has more difficulty reaching them

**Disadvantages:**

* Slower recovery
* More operational effort

---

### Local Backups

Stored at the same physical location as the primary system.

Fast to restore, but a physical disaster may destroy both the original and backup.

---

### Remote Backups

Stored at another location.

They provide better protection against:

* Fire
* Flood
* Theft
* Major site failures

A resilient backup strategy often combines these approaches.

---

# 13. Linux and Unix Security

Linux and Unix-like operating systems provide several mechanisms for system security.

A major characteristic is the use of **text-based configuration files**.

Many system-wide configuration files are stored under:

```text
/etc
```

Examples include configuration related to:

* Users
* Networking
* Services
* Authentication
* System behavior

Individual users may also have configuration files in their home directories.

Files beginning with a dot are commonly called **dot files**.

For example:

```text
~/.bashrc
~/.profile
```

These often contain user-specific configuration.

---

# 14. Linux File Permissions

Linux traditionally uses three primary permission categories:

```text
Owner
Group
Others
```

and three fundamental permissions:

```text
r = Read
w = Write
x = Execute
```

For example:

```text
-rwxr-x---
```

can be interpreted as:

```text
Owner  → rwx
Group  → r-x
Others → ---
```

This provides a basic mechanism for controlling access to files and directories.

---

# 15. Linux Security Threats

Linux systems can experience both **local** and **remote** attacks.

## Local Exploits

A local exploit is used by someone who already has some level of access to the system.

The attacker may attempt to escalate privileges:

```text
Low-Privilege User
       ↓
Local Vulnerability
       ↓
Privilege Escalation
       ↓
Root
```

---

## Remote Exploits

A remote exploit targets a service accessible over a network.

```text
Attacker
   ↓
Network
   ↓
Vulnerable Service
   ↓
Linux Server
```

Examples may include vulnerable:

* Web servers
* SSH services
* Database services
* Network applications

---

# 16. Linux Remote Access and Host Firewalls

Linux systems commonly use secure remote-access mechanisms such as **SSH**.

SSH provides encrypted remote administration and can be secured through:

* Strong authentication
* Key-based authentication
* Restricted users
* Limited privileges
* Firewall rules

Host-based firewall mechanisms can also restrict which network connections are allowed.

For example:

```text
Internet
   ↓
Host Firewall
   ↓
SSH → Allowed
Database → Blocked
Unused Services → Blocked
```

---

# 17. Windows Security

Windows provides several integrated security mechanisms for protecting systems and users.

Important components include:

* Windows Update
* WSUS
* Windows Registry
* Access Control
* Mandatory Integrity Control
* UAC
* BitLocker
* EFS

---

# 18. Windows Updates and WSUS

**Windows Update** provides automated operating-system updates.

Organizations can also use **Windows Server Update Services (WSUS)** to centrally manage updates in enterprise environments.

A simplified architecture is:

```text
Microsoft Updates
       ↓
      WSUS
       ↓
 ┌─────┼─────┐
 ↓     ↓     ↓
PC 1  PC 2  Server
```

This allows organizations to control and coordinate update deployment.

---

# 19. Windows Registry

The **Windows Registry** is a centralized configuration database used by Windows and applications.

It stores configuration information relating to:

* Operating-system settings
* Applications
* Hardware
* Users
* Services

Because the Registry contains important system configuration, unauthorized modification can have significant consequences.

---

# 20. Windows Access Control and Mandatory Integrity Control

Windows supports **Discretionary Access Control (DAC)** for controlling access to objects.

Windows also implements **Mandatory Integrity Control (MIC)**.

MIC assigns integrity levels to processes and objects.

Typical integrity levels include:

```text
Low
 ↓
Medium
 ↓
High
 ↓
System
```

The basic concept resembles the **Biba integrity model**, where information and processes are controlled according to their integrity levels.

For example, a low-integrity process should not normally be able to modify high-integrity resources.

---

# 21. User Account Control

**User Account Control (UAC)** helps prevent applications from automatically obtaining administrative privileges.

The idea is:

```text
Normal User Activity
       ↓
Standard Privileges
       ↓
Administrative Action Required
       ↓
Explicit Approval
       ↓
Elevated Privileges
```

This reduces the amount of time that administrative privileges are actively available.

UAC therefore supports the principle of **least privilege**.

---

# 22. Windows Encryption

Windows provides multiple encryption technologies.

### BitLocker

**BitLocker** provides full-volume or full-disk encryption.

It helps protect information if a storage device is physically removed from the computer.

```text
Hard Drive
     ↓
BitLocker
     ↓
Encrypted Storage
```

BitLocker uses modern cryptographic algorithms, including **AES** configurations.

---

### Encrypting File System

**Encrypting File System (EFS)** provides file-level encryption within supported Windows environments.

The key distinction is:

```text
BitLocker → Protects the storage volume
EFS       → Protects individual files
```

These mechanisms address different protection requirements.

---

# 23. Windows Security Baselines

Security baselines provide recommended configurations for Windows systems.

Historically, **Microsoft Baseline Security Analyzer (MBSA)** was used to identify certain security and configuration issues.

The broader principle remains important:

> Systems should be compared against a known secure configuration baseline.

A baseline can help identify:

* Missing updates
* Weak configurations
* Unnecessary services
* Security settings that do not meet organizational requirements

---

# 24. Virtualization

**Virtualization** allows multiple virtual systems to operate on shared physical hardware.

A **hypervisor** manages the underlying resources and provides virtual hardware to guest operating systems.

```text
Physical Hardware
       │
       ▼
   Hypervisor
   ┌────┼────┐
   ↓    ↓    ↓
  VM1  VM2  VM3
```

Each virtual machine can run its own operating system.

---

# 25. Hypervisor Responsibilities

The hypervisor acts as a resource broker between virtual machines and physical hardware.

It manages resources such as:

* CPU
* Memory
* Storage
* Network interfaces
* Virtual disks

It also performs functions such as:

* Starting and stopping VMs
* Managing VM lifecycle
* Providing virtual hardware
* Device emulation
* Resource allocation
* VM administration

Conceptually:

```text
Physical CPU
     ↓
Hypervisor
 ┌───┼───┐
 ↓   ↓   ↓
VM1 VM2 VM3
```

---

# 26. Software-Defined Networks

**Software-Defined Networking (SDN)** separates network control from traditional physical network infrastructure and allows networks to be managed programmatically.

Virtual networks can be created over existing physical networks.

For example:

```text
Physical Network
────────────────────────
        ↓
   Overlay Network
        ↓
Virtual Network Segments
```

Technologies such as **VXLAN** can be used to create logical overlay networks.

Security services can then be dynamically positioned within these virtual environments.

For example:

```text
Virtual Network
      │
      ├── Virtual Firewall
      │
      ├── Virtual IDS
      │
      └── Virtual Servers
```

This provides greater flexibility than relying exclusively on fixed physical security appliances.

---

# 27. Containers

Containers provide another form of application virtualization.

Unlike traditional virtual machines, containers generally share the host operating system's kernel.

```text
Physical Hardware
       ↓
Host OS Kernel
   ┌───┼───┐
   ↓   ↓   ↓
 C1   C2   C3
```

Each container provides an isolated application environment while sharing the underlying kernel.

---

## 27.1 Containers vs. Virtual Machines

A traditional VM generally includes its own guest operating system:

```text
Hypervisor
 ├── VM + Guest OS
 ├── VM + Guest OS
 └── VM + Guest OS
```

Containers share the host kernel:

```text
Host OS Kernel
 ├── Container
 ├── Container
 └── Container
```

Therefore, containers generally have:

* Lower overhead
* Faster startup
* Higher application density

However, sharing the host kernel means that a kernel-level vulnerability can potentially have consequences for multiple containers.

---

# 28. Virtualization Security

Virtual environments introduce security requirements that do not disappear simply because systems are virtual.

Important security concerns include:

* Guest OS isolation
* Hypervisor security
* Protection of VM images
* Protection of snapshots
* Virtual network security
* Administrative access
* VM-to-VM communication

A compromised virtual machine should not automatically be able to compromise other virtual machines.

---

# 29. Hypervisor Hardening

The hypervisor is a particularly important security component because it controls the virtual infrastructure.

It should therefore be treated similarly to a highly critical operating system.

Hardening should include:

* Security patching
* Strong authentication
* Minimal services
* Access control
* Security monitoring
* Secure configuration
* Regular security assessments

A useful principle is:

> **If the hypervisor is compromised, the security of the virtual machines may also be affected.**

---

# 30. Isolated Management Networks

Administrative traffic should ideally be separated from normal production traffic.

For example:

```text
                 Physical Network
                       │
          ┌────────────┴────────────┐
          │                         │
    Production Network       Management Network
          │                         │
       VMs/Apps                Hypervisor Admin
```

This makes it more difficult for an attacker who compromises a normal workload to directly reach hypervisor management interfaces.

---

# 31. Virtual Firewalls

Firewalls can also operate inside virtualized environments.

There are three important approaches.

---

## 31.1 VM Bastion Host

A dedicated virtual machine can run traditional security software.

For example:

```text
Virtual Network
      │
      ▼
┌──────────────┐
│ Firewall VM  │
│ IDS / IPS VM │
└──────┬───────┘
       │
       ▼
   Other VMs
```

The dedicated VM acts as a security gateway or bastion host.

---

## 31.2 VM Host-Based Firewall

Each guest operating system can run its own firewall.

```text
VM 1 → Host Firewall
VM 2 → Host Firewall
VM 3 → Host Firewall
```

This provides security directly at each workload.

For example, one server might allow:

```text
HTTPS → Allowed
SSH   → Restricted
Other → Blocked
```

while another server can have completely different rules.

---

## 31.3 Hypervisor Firewall

A firewall can also be integrated directly into the virtualization platform or hypervisor layer.

```text
Physical Hardware
       ↓
Hypervisor Firewall
       ↓
 ┌─────┼─────┐
 VM1  VM2   VM3
```

This allows traffic between virtual machines or virtual networks to be inspected at the virtualization layer.

---

# 32. Virtualization Security Architecture

A comprehensive virtual security architecture can combine multiple controls:

```text
                    Internet
                       │
                       ▼
                Physical Firewall
                       │
                       ▼
              Virtual Network
                       │
              ┌────────┴────────┐
              │                 │
       Virtual Firewall       Virtual IDS
              │                 │
              └────────┬────────┘
                       │
                   Hypervisor
                ┌──────┼──────┐
                │      │      │
               VM1    VM2    VM3
                │      │      │
              Host   Host   Host
            Firewall Firewall Firewall
```

This illustrates the principle of **defense in depth within virtual environments**.

Security must therefore cover not only the guest operating systems but also the **hypervisor, virtual networks, VM images, management interfaces, and supporting infrastructure**.

---

# 33. Security Across the System Lifecycle

The concepts discussed throughout this section can be viewed as one continuous process:

```text
          Security Planning
                 ↓
          Risk Assessment
                 ↓
          Secure Deployment
                 ↓
          OS Hardening
                 ↓
        Application Hardening
                 ↓
       Network & Access Control
                 ↓
       Monitoring & Logging
                 ↓
        Patch Management
                 ↓
       Backup & Recovery
                 ↓
        Incident Response
                 ↓
          Security Review
                 ↺
```

Whether the system is a traditional Linux server, Windows workstation, virtual machine, SDN environment, or container platform, the fundamental principle remains the same: **security must be designed into the system, continuously maintained, and regularly reassessed rather than treated as a one-time configuration task.**
