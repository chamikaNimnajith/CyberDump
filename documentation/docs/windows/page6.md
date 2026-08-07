# Windows Services: Architecture, Management, and Security Concepts

## Introduction

Windows services are an important part of the Windows operating system. They allow programs to run in the background without requiring a user to manually start them.

From a security perspective, services are extremely important because:

* They often run automatically.
* They may have high privileges.
* Misconfigured services can allow privilege escalation.
* Attackers commonly examine services after gaining access to a Windows system.

This note explains:

* What Windows services are
* How to manage services
* How services work internally
* Service privilege concepts
* Service security issues
* Creating custom Windows services

---

# 1. What Are Windows Services?

## Background Processes

A Windows service is a special type of process that runs in the background.

Unlike normal applications:

* No graphical interface is required.
* Users do not need to manually start them.
* They can run continuously.

Examples:

* Windows Update service
* Print Spooler
* Antivirus services
* Database services

---

# Windows Services vs Linux Daemons

Linux has a similar concept called:

**Daemons**

Examples:

```bash
sshd
cron
apache2
```

Windows equivalent:

```text
Services
```

Comparison:

| Linux     | Windows                 |
| --------- | ----------------------- |
| Daemon    | Service                 |
| systemd   | Service Control Manager |
| systemctl | sc.exe / PowerShell     |

---

# Service Startup Behavior

Many services are configured to start automatically during system boot.

Example:

```
Computer Starts
        |
        ↓
Windows Loads Kernel
        |
        ↓
Service Control Manager Starts Services
        |
        ↓
User Login
```

Some services start before any user logs in.

---

# Service Accounts

Every service must run under a user account.

This account determines:

* Permissions
* Resources the service can access
* Security privileges

Common Windows service accounts:

---

## Local Service

A low-privileged account.

Used when the service does not require many permissions.

Example:

```
Local Service
      |
      ↓
Limited System Access
```

---

## Network Service

Designed for services that require network communication.

It has:

* Limited local privileges
* Network identity

---

## Local System (SYSTEM)

The most powerful local account.

Also known as:

```
NT AUTHORITY\SYSTEM
```

SYSTEM has almost complete control over the machine.

It can:

* Access all files
* Modify system settings
* Control services
* Manage security policies

Example:

```
SYSTEM
  |
  |
  ↓
Full Local Machine Control
```

---

# Why Services Are Security Targets

Services are common privilege escalation targets because of:

## 1. Weak File Permissions

Example:

A service runs:

```
C:\Program Files\App\service.exe
```

but a normal user can modify it.

Attack scenario:

```
User modifies service.exe
        |
        ↓
Service restarts
        |
        ↓
SYSTEM executes modified file
```

---

## 2. Incorrect Service Permissions

A low-privileged user may have permission to:

* Modify service settings
* Start/stop services
* Change executable paths

---

## 3. Vulnerable DLL Loading

Services often load DLL files.

If Windows searches the wrong location, an attacker may place a malicious DLL.

This is called:

**DLL Hijacking**

---

# 2. Managing Services Using GUI

Windows provides a graphical service manager.

---

# Opening Service Control Manager

Options:

Run:

```
services.msc
```

or search:

```
Services
```

---

# Service Manager Interface

The GUI displays:

## Service Name

Technical identifier.

Example:

```
wuauserv
```

---

## Display Name

Human-readable name.

Example:

```
Windows Update
```

---

## Status

Shows:

* Running
* Stopped

---

## Startup Type

Defines when the service starts.

Options:

| Type          | Meaning               |
| ------------- | --------------------- |
| Automatic     | Starts with Windows   |
| Manual        | Starts when required  |
| Disabled      | Cannot start          |
| Trigger Start | Starts after an event |

---

## Log On Account

Shows which user runs the service.

Example:

```
Local System
```

or:

```
Network Service
```

---

# Limitation of GUI

The GUI is useful locally.

However, during a remote shell session:

Example:

```
Attacker
    |
    ↓
Reverse Shell
    |
    ↓
CMD Access
```

The attacker usually cannot open graphical tools.

Therefore, command-line service management is required.

---

# 3. Managing Services Using PowerShell

PowerShell provides powerful service enumeration commands.

---

# Get-Service

Lists Windows services.

Command:

```powershell
Get-Service
```

Example output:

```
Status     Name

Running    wuauserv
Stopped    Fax
```

---

# Selecting Specific Properties

PowerShell can filter output.

Example:

```powershell
Get-Service | Select-Object DisplayName, Status, ServiceName, CanStop
```

Displays:

| Property    | Meaning                          |
| ----------- | -------------------------------- |
| DisplayName | Friendly service name            |
| Status      | Running/Stopped                  |
| ServiceName | Internal name                    |
| CanStop     | Whether current user can stop it |

---

# Finding Service Executables

One of the most important enumeration steps is finding:

> Which executable file belongs to a service?

Command:

```powershell
Get-CimInstance -ClassName Win32_Service
```

Example output:

```
Name:
VMTools

Path:
C:\Program Files\VMware\vmtools.exe
```

---

# Finding Running Services Only

Command:

```powershell
Get-CimInstance -ClassName Win32_Service -Filter "State='Running'"
```

This shows:

* Running services
* Executable paths
* Service accounts

---

# Why Binary Paths Matter

Security professionals check service paths because they may reveal:

* Weak permissions
* Third-party software
* Misconfigured services

Example:

```
Service:

backup.exe

Runs as:

SYSTEM

Location:

C:\Backup\backup.exe
```

If users can modify:

```
backup.exe
```

it may become a privilege escalation opportunity.

---

# 4. Managing Services Using Command Prompt (sc.exe)

Windows includes:

```
sc.exe
```

(Service Control)

It allows complete service management.

---

# Listing Services

Command:

```cmd
sc.exe query
```

Shows:

* Service names
* Status
* Type
* Exit codes

---

# Starting and Stopping Services

Stop:

```cmd
sc.exe stop service_name
```

Example:

```cmd
sc.exe stop wpnservice
```

---

Start:

```cmd
sc.exe start service_name
```

---

# Permission Requirement

Most service operations require administrator privileges.

Example:

Normal user:

```
Access is denied
```

Administrator:

```
SUCCESS
```

---

# Viewing Service Configuration

Command:

```cmd
sc.exe qc service_name
```

Example output:

```
BINARY_PATH_NAME:
C:\Program Files\App\service.exe

START_TYPE:
AUTO_START

SERVICE_START_NAME:
LocalSystem
```

Important information:

* Executable location
* Startup configuration
* Running account

---

# Service Binary Path Hijacking

A service stores the executable it runs.

Example:

Original:

```
C:\Program Files\App\service.exe
```

An attacker changes it:

```
C:\Temp\malicious.exe
```

Command:

```cmd
sc.exe config service_name binpath= "C:\Temp\malicious.exe"
```

When the service starts:

```
Service Starts
       |
       ↓
Windows Executes New Binary
       |
       ↓
Code Runs With Service Privileges
```

If the service runs as SYSTEM:

```
SYSTEM Access
```

---

# Viewing Service Permissions

Command:

```cmd
sc.exe sdshow service_name
```

Output:

```
SDDL String
```

Example:

```
D:(A;;CCLCSWLOCRRC;;;SY)
```

This represents:

**Security Descriptor Definition Language (SDDL)**

It defines:

* Who can control the service
* Allowed actions
* Denied actions

---

# Modifying Service Permissions

Command:

```cmd
sc.exe sdset service_name SDDL_STRING
```

This changes service access permissions.

---

# 5. Creating a Custom Windows Service in C

A normal executable cannot automatically become a Windows service.

A service must communicate with:

```
Service Control Manager (SCM)
```

---

# Windows Service Architecture

The structure:

```
Service Control Manager

        |
        ↓

Service Program

        |
        ↓

ServiceMain()

        |
        ↓

Main Service Loop
```

---

# Required Windows APIs

A C service usually includes:

```c
#include <windows.h>
#include <stdio.h>
#include <stdlib.h>
```

The Windows API provides:

* Service registration
* Status updates
* Control handling

---

# Service Components

## 1. Main Entry Point

Function:

```
main()
```

or:

```
wWinMain()
```

Registers the service.

It uses:

```
StartServiceCtrlDispatcher()
```

to connect with Windows Service Manager.

---

# 2. ServiceMain()

The main service function.

Responsibilities:

* Initialize service
* Register control handler
* Set service status
* Start execution loop

---

# 3. Control Handler

Windows sends commands:

Examples:

* Stop
* Pause
* Continue
* Shutdown

The handler receives these requests.

Function:

```
SetServiceStatus()
```

updates the current service state.

---

# 4. Main Service Loop

A service normally runs continuously.

Example:

```
while(service_running)
{
    perform_task();

    sleep();
}
```

The example service:

* Waits
* Writes logs
* Continues running

---

# Logging Example

The service writes:

```
temp_logger.log
```

inside:

```
C:\Windows\Temp
```

Example:

```
Service Running

        ↓

Write log message

        ↓

Continue Loop
```

---

# 6. Compiling, Installing, and Removing a Service

## Step 1: Cross Compile

The service is compiled on Linux using:

```
mingw-w64
```

Example:

```bash
x86_64-w64-mingw32-gcc service.c -o simple_service.exe
```

Result:

```
simple_service.exe
```

---

# Step 2: Transfer File

The executable is moved to Windows.

Example:

```
Linux Machine

        ↓

simple_service.exe

        ↓

Windows Machine
```

---

# Step 3: Create Service

Run as Administrator:

```cmd
sc.exe create simple_service binpath= "C:\path\simple_service.exe"
```

Windows registers:

```
simple_service
```

inside the Service Control Manager.

---

# Step 4: Start Service

Command:

```cmd
sc.exe start simple_service
```

Windows launches:

```
simple_service.exe
```

as a service.

---

# Step 5: Verify Service

Check:

```cmd
sc.exe query simple_service
```

or open:

```
services.msc
```

The service should appear:

```
Running
```

The log file:

```
C:\Windows\Temp\temp_logger.log
```

should contain service activity.

---

# Removing the Service

Stop:

```cmd
sc.exe stop simple_service
```

Delete:

```cmd
sc.exe delete simple_service
```

This removes the service registration.

---

# NSSM Alternative

Sometimes developers have normal executables that were not designed as services.

A tool called:

```
NSSM
```

(Non-Sucking Service Manager)

can wrap normal programs and run them as Windows services.

Example:

```
Normal Application

        +

NSSM

        ↓

Windows Service
```

---

# Summary

Windows services are powerful background processes that control many operating system functions.

Important concepts:

| Concept                 | Explanation                                   |
| ----------------------- | --------------------------------------------- |
| Windows Service         | Background process managed by Windows         |
| Service Control Manager | Component that starts and manages services    |
| SYSTEM Account          | Highest local privilege account               |
| Get-Service             | PowerShell service enumeration                |
| sc.exe                  | Command-line service management tool          |
| Service Binary Path     | Executable file launched by service           |
| Binary Path Hijacking   | Changing service executable to gain execution |
| SDDL                    | Service permission description format         |
| Custom Service          | Program designed to run under SCM             |

Security professionals examine services because:

```
Misconfigured Service
        |
        ↓
Executable Modification
        |
        ↓
Service Restart
        |
        ↓
Higher Privilege Execution
```

Understanding Windows services is essential for Windows administration, privilege escalation, malware analysis, and penetration testing.
