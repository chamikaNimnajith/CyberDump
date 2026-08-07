# Windows Registry Abuse for Privilege Escalation and Persistence

## 1. Windows Registry Basics

### What is the Windows Registry?

The **Windows Registry** is a centralized database used by the Windows operating system to store configuration settings and information about:

* Operating system components
* Hardware devices
* Installed applications
* User preferences
* Security settings
* Services and startup programs

Instead of storing configuration information in many separate files, Windows stores important settings inside the Registry.

Think of the Registry as a **configuration database for Windows**.

---

## Registry Structure

The Windows Registry uses a **tree-like hierarchical structure**.

It contains:

* **Hives**
* **Keys**
* **Subkeys**
* **Values**

Example:

```
HKEY_LOCAL_MACHINE
        |
        └── SYSTEM
              |
              └── CurrentControlSet
                    |
                    └── Services
                          |
                          └── ServiceName
                                |
                                └── ImagePath
```

### Registry Hive

A **Hive** is the top-level section of the Windows Registry.

Each hive represents a major category of Windows configuration.

The five main registry hives are:

| Hive                | Abbreviation | Purpose                                           |
| ------------------- | ------------ | ------------------------------------------------- |
| HKEY_CLASSES_ROOT   | HKCR         | Stores file associations and application settings |
| HKEY_LOCAL_MACHINE  | HKLM         | Stores system-wide configuration                  |
| HKEY_USERS          | HKU          | Stores settings for all user accounts             |
| HKEY_CURRENT_USER   | HKCU         | Stores settings for the currently logged-in user  |
| HKEY_CURRENT_CONFIG | HKCC         | Stores current hardware configuration             |

---

## Registry Hive Files

Although the Registry appears as a single database, its data is stored physically on disk.

Each hive is saved as a separate file.

Common locations:

```
C:\Windows\System32\Config\
```

Examples:

| Registry Hive | File     |
| ------------- | -------- |
| HKLM\SYSTEM   | SYSTEM   |
| HKLM\SOFTWARE | SOFTWARE |
| HKLM\SECURITY | SECURITY |
| HKLM\SAM      | SAM      |
| HKU.DEFAULT   | DEFAULT  |

User-specific registry data is stored inside:

```
C:\Users\<username>\NTUSER.DAT
```

---

## Why is the Registry Important in Security?

The Windows Registry is an important source of **threat intelligence**.

Security analysts monitor registry activity because attackers often modify registry keys to:

* Maintain persistence
* Execute malicious programs
* Escalate privileges
* Disable security tools
* Modify system behavior

Registry modifications can become **Indicators of Compromise (IOCs)**.

Example:

A normal system:

```
HKCU\Software\Microsoft\Windows\CurrentVersion\Run
```

contains:

```
OneDrive.exe
```

A suspicious system:

```
HKCU\Software\Microsoft\Windows\CurrentVersion\Run
```

contains:

```
malware.exe
```

This indicates possible malware persistence.

---

# Registry Attack Model

The main security question is:

> "What can an attacker do if they can modify important registry values?"

If an attacker gains permission to modify sensitive registry keys, they may achieve:

### 1. Arbitrary Code Execution

An attacker can change a registry value to execute their own program.

Example:

Changing:

```
ImagePath = C:\Windows\Service.exe
```

to:

```
ImagePath = C:\Users\Attacker\malware.exe
```

When Windows starts the service, malware executes.

---

### 2. Privilege Escalation

Registry keys controlling privileged processes can allow attackers to execute code with higher privileges.

Example:

A service running as:

```
NT AUTHORITY\SYSTEM
```

can execute an attacker-controlled program if its registry configuration is modified.

---

### 3. Persistence

Attackers can modify startup registry keys so their malware runs every time a user logs in.

Example:

```
HKCU\Software\Microsoft\Windows\CurrentVersion\Run
```

---

# 2. Service ImagePath Registry Abuse

## Windows Services Overview

Windows Services are background programs that run automatically.

Examples:

* Windows Update service
* Antivirus services
* Database services

A service configuration contains:

* Service name
* Startup type
* Account used to run the service
* Executable path

Example:

```
Service Name:
Spooler

Executable:
C:\Windows\System32\spoolsv.exe
```

---

# Where Service Paths Are Stored

When we configure a service using:

```
sc.exe
```

we see:

```
binPath
```

Example:

```
sc qc ServiceName
```

Output:

```
BINARY_PATH_NAME : C:\Program Files\Service\service.exe
```

However, Windows stores this information internally in the Registry.

Location:

```
HKLM\System\CurrentControlSet\Services\<ServiceName>
```

The executable path is stored in:

```
ImagePath
```

Example:

```
HKLM\System\CurrentControlSet\Services\TestService

ImagePath = C:\service\hello.exe
```

---

# Service ImagePath Attack

If an attacker can modify the service registry key, they can replace the legitimate executable with their own program.

Example:

Original:

```
ImagePath =
C:\Program Files\Service\Service.exe
```

Modified:

```
ImagePath =
C:\Users\Attacker\hello.exe
```

When the service starts:

```
Start-Service ServiceName
```

Windows executes:

```
hello.exe
```

instead of the original service.

---

## Example Attack Flow

1. Attacker finds a vulnerable service.

2. Attacker modifies:

```
HKLM\System\CurrentControlSet\Services\ServiceName
```

3. Changes:

```
ImagePath
```

4. Starts the service.

5. Malicious executable runs.

---

## Difference From Weak Service Permissions

This attack is different from traditional service permission abuse.

### Weak Service Permission Attack

Attacker modifies:

* Service configuration
* Service binary path
* Startup settings

using service management tools.

Example:

```
sc.exe config
```

---

### Registry ImagePath Attack

Attacker directly modifies:

```
Registry Key
        |
        └── ImagePath value
```

Windows reads the modified value and executes the attacker-controlled program.

---

# 3. AppInit_DLLs Registry Abuse

## What is AppInit_DLLs?

AppInit_DLLs is a Windows feature that automatically loads DLL files into user applications.

Registry location:

```
HKLM\Software\Microsoft\Windows NT\CurrentVersion\Windows
```

The purpose was originally to allow software developers to load additional libraries into applications.

---

# How It Works

When a graphical Windows application starts:

```
Application.exe
        |
        |
        ↓
Windows loads DLLs
        |
        |
        ↓
AppInit_DLLs DLL is loaded
```

This means one DLL can be loaded into many applications automatically.

---

# Security Risk

Attackers can abuse this feature by inserting a malicious DLL.

Example:

```
AppInit_DLLs =
C:\Malware\evil.dll
```

Every compatible GUI application starts loading:

```
evil.dll
```

This provides:

* Code execution
* Malware injection
* Persistence

---

# Important Registry Values

## LoadAppInit_DLLs

Controls whether the feature is enabled.

Enabled:

```
LoadAppInit_DLLs = 1
```

Disabled:

```
LoadAppInit_DLLs = 0
```

---

## AppInit_DLLs

Stores DLL locations.

Example:

```
AppInit_DLLs =
C:\Tools\monitor.dll;
C:\Tools\security.dll
```

Multiple DLLs are separated using:

```
;
```

---

# Modern Windows Protection

Because attackers abused AppInit_DLLs, Microsoft reduced its usage.

Modern Windows versions:

* Disable it by default
* Require additional security conditions
* Limit unsigned DLL loading

Default:

```
LoadAppInit_DLLs = 0
```

---

# 4. Run and RunOnce Registry Keys

## Purpose

The **Run** and **RunOnce** registry keys control programs that automatically start when a user logs in.

They are commonly abused by malware for persistence.

---

# Registry Locations

## Current User Startup

```
HKCU\Software\Microsoft\Windows\CurrentVersion\Run
```

Affects:

* Only the current user

Example:

```
HKCU\...\Run

Updater = C:\Users\User\malware.exe
```

---

## All Users Startup

```
HKLM\Software\Microsoft\Windows\CurrentVersion\Run
```

Affects:

* Every user on the machine

Requires:

* Administrator privileges

---

# Run vs RunOnce

## Run

Programs execute every time the user logs in.

Example:

```
Run

Discord.exe
OneDrive.exe
malware.exe
```

---

## RunOnce

Programs execute only once.

After execution:

* Windows removes the registry entry.

Example:

```
RunOnce

Setup.exe
```

Execution:

```
Login
 ↓
Setup.exe runs
 ↓
Registry entry deleted
```

---

# Persistence Attack Example

Attacker creates:

```
HKCU\Software\Microsoft\Windows\CurrentVersion\Run
```

Entry:

```
Updater =
C:\Users\User\hello.exe
```

Next login:

```
User logs in
        |
        ↓
Windows checks Run keys
        |
        ↓
hello.exe automatically executes
```

---

# 5. Winlogon Registry Abuse

## What is Winlogon?

**Winlogon** is a Windows component responsible for the login process.

It controls what happens after a user successfully authenticates.

Registry location:

```
HKLM\Software\Microsoft\Windows NT\CurrentVersion\Winlogon
```

---

# Important Winlogon Values

## Shell

Defines the Windows graphical shell.

Default:

```
Shell = explorer.exe
```

Explorer provides:

* Desktop
* Taskbar
* Start Menu
* File Explorer

---

## Userinit

Runs after login.

Default:

```
Userinit =
C:\Windows\System32\userinit.exe
```

Responsible for initializing the user environment.

---

## UIHost

Controls the Windows login interface.

Example:

```
UIHost =
SIHost.exe
```

---

# Shell Hijacking Attack

If an attacker changes:

```
Shell
```

from:

```
explorer.exe
```

to:

```
cmd.exe
```

Windows behavior changes.

Normal login:

```
Login
 |
 ↓
explorer.exe
 |
 ↓
Desktop appears
```

After modification:

```
Login
 |
 ↓
cmd.exe
 |
 ↓
Command Prompt only
```

The user sees:

```
C:\>
```

instead of the Windows desktop.

---

# Why Attackers Use Shell Hijacking

Shell hijacking can provide:

* Persistence
* Command execution after login
* Control over user sessions

It can also be used to hide malware execution.

---

# Restoring the System

An administrator can restore the original shell:

Registry:

```
HKLM\Software\Microsoft\Windows NT\CurrentVersion\Winlogon
```

Change:

```
Shell = cmd.exe
```

Back to:

```
Shell = explorer.exe
```

After logging in again:

```
Windows Desktop loads normally
```

---

# Summary Table

| Registry Technique | Registry Location                                         | Purpose                      | Attack Result                         |
| ------------------ | --------------------------------------------------------- | ---------------------------- | ------------------------------------- |
| Service ImagePath  | HKLM\System\CurrentControlSet\Services                    | Controls service executable  | Code execution / privilege escalation |
| AppInit_DLLs       | HKLM\Software\Microsoft\Windows NT\CurrentVersion\Windows | Loads DLLs into applications | DLL injection / persistence           |
| Run Keys           | HKCU/HKLM...\Run                                          | Startup programs             | Malware persistence                   |
| RunOnce            | HKCU/HKLM...\RunOnce                                      | One-time startup execution   | Temporary execution                   |
| Winlogon Shell     | HKLM...\Winlogon                                          | Controls login shell         | Shell hijacking                       |

---

## Key Security Concept

The Windows Registry is a powerful configuration system.
Because many critical Windows components depend on registry values, **unauthorized registry modification can allow attackers to execute code, gain higher privileges, and maintain persistence on a compromised system.**

Security teams monitor registry changes because they often reveal attacker activity.
