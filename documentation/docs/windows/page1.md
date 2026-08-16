# Windows Shells

## Introduction

When learning Windows security, privilege escalation, or penetration testing, it is important to first understand how Windows is designed and how users, permissions, and system components work. These notes explain the basic concepts in a simple and beginner-friendly way.

---

# 1. Windows vs. Linux: Architecture & Design

Before learning Windows permissions, it's useful to understand how Windows differs from Linux.

## Open Source vs Proprietary

### Linux

* Linux is **open source**.
* Created by **Linus Torvalds** in 1991.
* Anyone can view, study, or modify the source code.
* Developers around the world contribute improvements.

Because everything is public, security researchers can understand how Linux works internally.

### Windows

Windows is **proprietary software** developed by Microsoft.

This means:

* Source code is not publicly available.
* Internal implementation is mostly hidden.
* Microsoft controls updates and development.

Therefore, researchers usually understand Windows through documentation, reverse engineering, and experimentation.

---

## Operating System Architecture

Every operating system has three major layers.

```
Applications
       │
System Calls
       │
Kernel
       │
Hardware
```

### Hardware

The physical components of a computer:

* CPU
* RAM
* Hard drive
* Network card
* Keyboard
* Display

---

### Kernel

The **kernel** is the heart of the operating system.

Its responsibilities include:

* Managing memory
* Managing processes
* Communicating with hardware
* Managing users and permissions
* Managing files
* Handling networking

Applications cannot access hardware directly.

Instead, they request services from the kernel using **system calls**.

For example:

When a program wants to save a file:

```
Application
      ↓
System Call
      ↓
Kernel
      ↓
Hard Drive
```

The kernel performs the operation safely.

---

### User Space (Applications)

Applications include:

* Chrome
* Firefox
* Microsoft Word
* VS Code
* Notepad

These programs run in **user mode** and must ask the kernel whenever they need hardware access.

---

## Linux Distributions vs Windows Versions

### Linux

The Linux kernel can be combined with different software collections.

These are called **distributions (distros).**

Examples:

* Ubuntu
* Debian
* Kali Linux
* Arch Linux
* Fedora

Each distribution shares the same Linux kernel but offers different software and configurations.

---

### Windows

Windows does **not** have distributions.

Microsoft provides the kernel and userland together as one product.

Instead of distributions, Windows has **versions**, such as:

* Windows XP
* Windows 7
* Windows 8
* Windows 10
* Windows 11

For servers:

* Windows Server 2016
* Windows Server 2019
* Windows Server 2022

---



# 3. Understanding Windows Shells

Many beginners confuse the terms **terminal** and **shell**.

They are not the same.

---

## Terminal

The terminal is the application where you type commands.

Examples:

* Windows Terminal
* GNOME Terminal

Think of it as the **window** you interact with.

---

## Shell

The shell is the command interpreter.

It:

* Reads commands
* Understands them
* Sends requests to the kernel

---

## Windows Has Two Main Shells

### Command Prompt (CMD)

Executable:

```
cmd.exe
```

Characteristics:

* Older shell
* Based on MS-DOS
* Simple commands
* Mostly used for compatibility

Example:

```cmd
dir
```

---

### PowerShell

Executable:

```
powershell.exe
```

PowerShell is much more powerful.

Features:

* Object-oriented output
* Advanced scripting
* Better automation
* Rich administrative capabilities

---

## Visual Difference

### CMD

```
C:\Users\Admin>
```

### PowerShell

```
PS C:\Users\Admin>
```

The **PS** prefix tells you that PowerShell is running.

---

## Switching Between Shells

From CMD:

```cmd
powershell
```

Sometimes:

```cmd
powershell -ep bypass
```

This bypasses PowerShell execution policy restrictions for the current session.

Back to CMD:

```powershell
cmd
```

---

# 4. Useful Command Prompt (CMD) Commands

## A. Basic System Commands

### systeminfo

Displays information about the system.

```cmd
systeminfo
```

Shows:

* Windows version
* Build number
* Hostname
* Processor
* RAM
* Domain information
* Boot time

---

### whoami

Displays the current logged-in user.

```cmd
whoami
```

Example:

```
DESKTOP\Admin
```

---

### cd

Shows or changes the current directory.

```cmd
cd
```

Example:

```
C:\Users\Admin
```

---

### set

Lists all environment variables.

```cmd
set
```

Common variables:

* PATH
* TEMP
* APPDATA
* USERPROFILE

---

### echo %VARIABLE%

Prints a specific environment variable.

Example:

```cmd
echo %PATH%
```

The **PATH** variable contains directories where Windows searches for executable programs.

---

### where

Finds the location of an executable.

```cmd
where python
```

Similar to Linux:

```bash
which python
```

---

### help

Displays help for a command.

```cmd
help dir
```

or

```cmd
dir /?
```

---

### cls

Clears the terminal screen.

```cmd
cls
```

---

# B. File System Commands

## dir

Lists files and folders.

```cmd
dir
```

---

## dir /a

Shows all files, including hidden ones.

```cmd
dir /a
```

---

## mkdir

Creates a directory.

```cmd
mkdir Projects
```

---

## Create an Empty File

```cmd
type nul > notes.txt
```

Creates a zero-byte file.

---

## Create a File with Text

```cmd
echo Hello > hello.txt
```

---

## Read a File

```cmd
type hello.txt
```

---

# C. Permission & Privilege Commands

These commands are especially important for security assessments and privilege escalation.

---

## whoami /groups

Displays all security groups the current user belongs to.

```cmd
whoami /groups
```

Examples:

* Users
* Administrators
* Remote Desktop Users

---

## whoami /priv

Shows user privileges.

```cmd
whoami /priv
```

Examples:

* SeShutdownPrivilege
* SeBackupPrivilege
* SeImpersonatePrivilege

One of the most important privileges is **SeImpersonatePrivilege**, which is commonly abused in Windows privilege escalation attacks.

---

## net accounts

Displays password policy information.

```cmd
net accounts
```

Shows:

* Minimum password length
* Password age
* Lockout threshold

---

## net user

Lists all users.

```cmd
net user
```

---

## net user username

Shows detailed information about a specific user.

```cmd
net user Administrator
```

Displays:

* Account status
* Group memberships
* Password settings

---

## icacls

Displays file permissions (ACLs).

```cmd
icacls secret.txt
```

Example output:

```
Users:(R)
Administrators:(F)
SYSTEM:(F)
```

Where:

* **F** = Full Control
* **M** = Modify
* **RX** = Read & Execute
* **R** = Read
* **W** = Write

Understanding ACLs is essential because they define **who can access a file or folder and what actions they are allowed to perform**.

---

# D. Network Commands

## ipconfig /all

Shows detailed network information.

```cmd
ipconfig /all
```

Displays:

* IP address
* MAC address
* DNS servers
* Gateway

---

## route print

Shows the routing table.

```cmd
route print
```

Useful for understanding how network traffic is routed.

---

## netstat -ano

Lists:

* Active connections
* Listening ports
* Process IDs (PIDs)

```cmd
netstat -ano
```

---

## TCP Only

```cmd
netstat -ano -p tcp
```

Shows only TCP connections.

---

# 5. Useful PowerShell Commands

PowerShell uses a consistent **Verb-Noun** naming style.

Examples:

* `Get-Process`
* `Get-Service`
* `Get-LocalUser`

This makes commands easier to remember.

---

## A. User & Group Commands

### Get Local Users

```powershell
Get-LocalUser
```

Lists all local user accounts.

---

### Get Local Groups

```powershell
Get-LocalGroup
```

Lists all local security groups.

---

### Group Members

```powershell
Get-LocalGroupMember Administrators
```

Shows every user in the **Administrators** group.

---

### Environment Variables

```powershell
dir env:
```

Lists all environment variables.

---

### Search for Files

```powershell
Get-ChildItem -Path C:\ -Filter *.txt -Recurse -ErrorAction SilentlyContinue
```

Useful for finding:

* Configuration files
* Password databases (e.g., `.kdbx`)
* Text files
* Logs

---

# B. Processes & Installed Software

## Running Processes

```powershell
Get-Process
```

Displays all running applications and background processes.

---

## Installed Applications

### 64-bit Programs

```powershell
Get-ItemProperty HKLM:\Software\Microsoft\Windows\CurrentVersion\Uninstall\*
```

### 32-bit Programs

```powershell
Get-ItemProperty HKLM:\Software\Wow6432Node\Microsoft\Windows\CurrentVersion\Uninstall\*
```

Security professionals often inspect installed third-party software because outdated applications are common sources of vulnerabilities and can provide opportunities for privilege escalation.

---

# C. Services

Windows **services** are background programs that start automatically or manually and continue running even when no user is logged in.

Examples include:

* Windows Update
* Print Spooler
* Antivirus services

In penetration testing, services are frequently examined because misconfigured service permissions or executable paths can lead to privilege escalation.

---

## List All Services

```powershell
Get-Service
```

---

## Show Only Running Services

```powershell
Get-Service | Where-Object {$_.Status -eq "Running"}
```

This displays only active services, helping administrators and security professionals quickly identify what is currently running.

---

# Summary

Understanding Windows architecture, shells, permissions, users, services, and networking commands forms the foundation of Windows administration and security. Commands such as `whoami /priv`, `icacls`, `net user`, `Get-LocalUser`, and `Get-Service` are especially valuable during system administration, incident response, and penetration testing because they reveal how Windows manages identities, permissions, and running services. Mastering these basics will make it much easier to progress into advanced topics such as Windows authentication, Access Control Lists (ACLs), Active Directory, and privilege escalation.
