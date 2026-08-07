# Antivirus Detection and Windows AMSI Security Mechanisms

## 1. Antivirus and Malware Detection Methodologies

## What is an Antivirus?

An **antivirus (AV)** is a security program designed to:

* Detect malware
* Prevent malicious execution
* Remove infected files
* Protect systems from cyber threats

Examples of malware detected by antivirus software:

* Viruses
* Trojans
* Ransomware
* Spyware
* Worms
* Malicious scripts

Modern antivirus solutions do not rely on a single detection method. They combine multiple techniques to identify suspicious files and behaviors.

---

# 1. Signature-Based Detection

## What is Signature Detection?

Signature-based detection works by comparing a file against a database of known malware patterns.

A **signature** is a unique identifier that represents known malicious code.

The simplest signature is a file hash.

Example:

```
malware.exe

SHA-256:
8f7e3b2a9c....
```

The antivirus checks:

```
File Hash
    |
    ↓
Known Malware Database
    |
    ↓
Match Found?
```

If the hash exists in the database:

```
Malware Detected
```

---

## Limitation of Hash-Based Detection

Hash signatures are fragile.

Even changing one byte changes the entire hash.

Example:

Original file:

```
malware.exe

SHA-256:
ABC123456
```

After adding one empty byte:

```
malware_modified.exe

SHA-256:
XYZ987654
```

The hashes are completely different.

The antivirus may no longer recognize it.

---

## Other Signature Types

Antivirus solutions also use more advanced signatures.

### String Signatures

Searching for suspicious text patterns.

Example:

```
powershell -enc
```

or:

```
CreateRemoteThread
VirtualAlloc
```

---

### Byte Sequence Signatures

Searching for specific machine-code patterns.

Example:

```
48 8B 05 XX XX XX
```

These patterns represent instructions commonly used by malware.

---

# 2. Static Analysis

## What is Static Analysis?

Static analysis examines a file **without executing it**.

The antivirus analyzes:

* File structure
* Imported DLLs
* Executable headers
* Strings
* Functions
* Code instructions

Example:

A security researcher opens:

```
malware.exe
```

inside:

* Ghidra
* IDA Pro
* Binary Ninja

and examines the code.

---

## What Does Static Analysis Look For?

### Imported Libraries

Example:

Suspicious imports:

```
CreateProcess()
VirtualAlloc()
WriteProcessMemory()
```

These APIs are commonly used for:

* Process injection
* Memory manipulation
* Code execution

---

### Entry Point

The antivirus checks where execution begins.

Example:

```
Executable Entry Point
        |
        ↓
Malicious Function
```

---

### Embedded Strings

Example:

A malware file contains:

```
cmd.exe
powershell.exe
http://malicious-site.com
```

This can indicate suspicious behavior.

---

# Packers and Static Analysis Problems

## What is a Packer?

A packer is a program that compresses or encrypts an executable.

Instead of storing:

```
Real Malware Code
```

the file contains:

```
Packed Wrapper
        |
        ↓
Encrypted Payload
```

When executed:

```
Program starts
       |
       ↓
Packer runs
       |
       ↓
Payload is unpacked in memory
       |
       ↓
Malware executes
```

---

## Why Attackers Use Packers

Packers make static analysis harder because:

* Original code is hidden
* Strings are encrypted
* File structure changes

The antivirus must analyze the unpacked version during runtime.

---

# 3. Dynamic Analysis

## What is Dynamic Analysis?

Dynamic analysis examines malware while it is running.

Instead of asking:

> "What does this file contain?"

it asks:

> "What does this file do?"

---

## Sandbox Environment

A **sandbox** is an isolated environment where suspicious programs can safely execute.

Example:

```
Suspicious File

      |
      ↓

Sandbox Machine

      |
      ↓

Monitor Behavior
```

The real computer remains protected.

---

## What Does a Sandbox Monitor?

### System Calls

Example:

```
CreateFile()
WriteFile()
CreateProcess()
```

---

### File System Activity

Example:

Malware creates:

```
C:\Windows\Temp\backdoor.exe
```

---

### Registry Changes

Example:

Creates persistence:

```
HKCU\Software\Microsoft\Windows\CurrentVersion\Run
```

---

### Network Communication

Example:

Malware connects to:

```
attacker-server.com
```

---

### Loaded APIs

Example:

Suspicious API usage:

```
VirtualAlloc()
CreateRemoteThread()
```

---

# 4. Machine Learning Detection

## What is ML-Based Detection?

Machine learning antivirus systems analyze many characteristics of files and determine whether they resemble malware.

Instead of checking:

```
Is this exact file malware?
```

ML asks:

```
Does this file look like malware?
```

---

## Features Used by ML Models

Examples:

### File Features

* File size
* Entropy
* Header information
* Imported libraries

### Behavioral Features

* Creates processes
* Modifies registry
* Contacts external servers

---

## Example

A file has:

```
High entropy
+
Suspicious API calls
+
Hidden sections
+
Network activity
```

The model may classify it as:

```
Malicious probability: 95%
```

---

# 2. Network and Enterprise Security Technologies

Antivirus is only one layer of security.

Modern organizations use multiple defensive systems.

---

# Firewalls

## What is a Firewall?

A firewall controls network traffic based on rules.

It analyzes:

* Source IP
* Destination IP
* Ports
* Protocols

Example:

Allow:

```
Client → Web Server
Port 443
```

Block:

```
Internet → Internal Database
Port 3306
```

---

# Web Application Firewall (WAF)

## What is a WAF?

A WAF protects web applications by inspecting HTTP-level traffic.

It understands:

* HTTP requests
* Cookies
* Parameters
* Headers

---

Example attack:

```
GET /login?id=' OR 1=1--
```

A WAF can detect:

```
SQL Injection Attempt
```

---

## Difference Between Firewall and WAF

| Firewall                 | WAF                          |
| ------------------------ | ---------------------------- |
| Network layer protection | Application layer protection |
| Checks IPs and ports     | Checks web requests          |
| Protects networks        | Protects web applications    |

---

# IDPS

## Intrusion Detection and Prevention System

IDPS monitors network activity for suspicious behavior.

Two types:

### IDS

Intrusion Detection System

* Detects attacks
* Generates alerts

Example:

```
Possible malware traffic detected
```

---

### IPS

Intrusion Prevention System

* Detects attacks
* Automatically blocks them

---

# EDR

## Endpoint Detection and Response

EDR focuses on individual devices.

Examples:

* Employee laptops
* Servers
* Workstations

EDR monitors:

* Processes
* Memory activity
* File changes
* Network connections

---

Example:

A user opens:

```
invoice.exe
```

EDR detects:

```
invoice.exe
   |
   ↓
PowerShell execution
   |
   ↓
Registry modification
```

and raises an alert.

---

# 3. Windows Security Suite and AMSI

## Windows Built-in Security Features

Windows includes several security technologies.

---

## Windows Defender Antivirus

Provides:

* Malware detection
* Real-time protection
* Threat removal

---

## Windows Firewall

Controls network communication.

---

## Windows Hello

Provides passwordless authentication using:

* Biometrics
* PIN
* Hardware security

---

## BitLocker

Provides:

* Full disk encryption
* Data protection

---

## Secure Boot

Ensures only trusted software loads during startup.

---

# AMSI (Anti-Malware Scan Interface)

## What is AMSI?

AMSI is a Microsoft security interface introduced in 2015.

Its purpose:

> Allow applications to communicate with antivirus software before executing content.

---

## Why Was AMSI Created?

Many attacks use scripting engines:

* PowerShell
* WMI
* VBScript
* JavaScript

Traditional antivirus may not see the actual script behavior.

AMSI allows these applications to submit content for scanning.

---

# AMSI Architecture

The relationship:

```
Application
(PowerShell)

       |
       |
       ↓

AMSI Interface

       |
       |
       ↓

Antivirus Provider
(Windows Defender)
```

---

# How AMSI Works

Example:

A user runs:

```powershell
malicious-script.ps1
```

Flow:

```
PowerShell receives script

        ↓

Loads amsi.dll

        ↓

Creates AMSI Context

        ↓

Sends script to AMSI

        ↓

AMSI sends request to Antivirus

        ↓

Antivirus gives verdict

        ↓

Allow or Block
```

---

# Important AMSI Functions

## AmsiScanBuffer

Scans data stored in memory.

Example:

```
Script Buffer
      |
      ↓
AmsiScanBuffer()
```

---

## AmsiScanString

Scans string content.

Example:

```
PowerShell command
      |
      ↓
AmsiScanString()
```

---

# 4. Conceptual AMSI Memory Manipulation

## Why Can AMSI Be Targeted?

When a program runs:

```
PowerShell.exe
```

Windows loads:

```
amsi.dll
```

into the same memory space.

Because the process owns its memory:

```
PowerShell Memory

+----------------+
| PowerShell     |
| amsi.dll       |
| AMSI Objects   |
+----------------+
```

the process can theoretically modify its own memory.

---

# AmsiContext Manipulation

## Normal Operation

The application creates:

```
AmsiContext
```

This connects the application to the antivirus provider.

Every scan request uses this context.

---

## Conceptual Attack

If the context is corrupted:

```
AmsiContext
      |
      X
      |
Antivirus Provider
```

The scan request may fail.

The application may continue execution because the antivirus was not successfully contacted.

---

# amsiInitFailed Manipulation

Another internal AMSI component:

```
amsiInitFailed
```

normally indicates whether AMSI initialization succeeded.

Normal:

```
amsiInitFailed = false
```

Meaning:

```
AMSI works
```

---

If changed:

```
amsiInitFailed = true
```

The application behaves as if:

```
AMSI initialization failed
```

and scanning may not continue.

---

# 5. Important Limitation: Memory vs Disk Protection

## AMSI Bypass Is Temporary

Memory manipulation only affects the current process.

Example:

```
PowerShell Session

AMSI modified

Commands execute
```

Close PowerShell:

```
Process destroyed
```

Start again:

```
New PowerShell
+
Fresh amsi.dll
+
Normal protection restored
```

---

# Disk-Based Detection Still Works

Even if a process avoids AMSI scanning, writing files triggers other security mechanisms.

Example:

Command:

```powershell
Out-File malware.ps1
```

creates:

```
malware.ps1
```

on disk.

Windows Defender monitors:

* File creation
* File modification
* Downloads

Flow:

```
Process writes file

        ↓

Filesystem event

        ↓

Antivirus scans file

        ↓

Threat detected
```

---

# Final Summary

| Technology          | Purpose                               |
| ------------------- | ------------------------------------- |
| Signature Detection | Matches known malware patterns        |
| Static Analysis     | Examines files without execution      |
| Dynamic Analysis    | Observes malware behavior in sandbox  |
| Machine Learning    | Classifies suspicious patterns        |
| Firewall            | Controls network traffic              |
| WAF                 | Protects web applications             |
| IDPS                | Detects/prevents network attacks      |
| EDR                 | Monitors endpoints                    |
| AMSI                | Connects applications with antivirus  |
| Memory Manipulation | Temporarily affects current process   |
| Disk Monitoring     | Detects malicious files independently |

---

## Key Security Concept

Modern malware detection is based on **multiple defensive layers**.

A single protection mechanism can fail, but combining:

* Antivirus
* AMSI
* EDR
* Firewalls
* Behavioral monitoring
* Machine learning

creates a stronger security system.

Understanding how these mechanisms work helps security professionals both **build better defenses** and **identify weaknesses attackers may attempt to exploit**.
