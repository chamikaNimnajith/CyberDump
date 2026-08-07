# Windows Scheduled Tasks 

## Overview

**Scheduled Tasks** are a Windows feature that allows programs, scripts, and commands to run automatically when a specific condition is met.

They are commonly used for:

* System maintenance
* Software updates
* Backups
* Automated scripts
* Application management

From a security perspective, scheduled tasks are important because they can execute programs automatically, sometimes with **high privileges**.

---

# 1. What are Scheduled Tasks?

A **Scheduled Task** is an automated job that runs when a specific trigger occurs.

The trigger can be:

* A specific time
* User login
* System startup
* A system event

Example:

```text
Every day at 2:00 AM
        |
        ▼
Run backup.exe
```

---

## Linux Equivalent

The Linux equivalent of Windows Scheduled Tasks is:

**Cron Jobs**

Example:

Linux:

```text
cron job
```

Windows:

```text
Scheduled Task
```

They both automate the execution of commands and scripts.

---

## Scheduled Tasks vs Windows Services

They are similar but have different purposes.

| Feature          | Scheduled Tasks                    | Windows Services                      |
| ---------------- | ---------------------------------- | ------------------------------------- |
| Purpose          | Run tasks at specific times/events | Run background processes continuously |
| Linux Equivalent | Cron jobs                          | Daemons                               |
| Trigger          | Time, login, events                | System startup                        |
| Example          | Backup script                      | Database service                      |

---

# 2. Why Are Scheduled Tasks Important for Security?

Scheduled tasks can execute:

* `.exe` files
* `.bat` scripts
* PowerShell scripts
* Other commands

Example:

```text id="5qz3lw"
Scheduled Task

        ↓

powershell.exe

        ↓

Execute Script
```

Because scheduled tasks can run code automatically, they become a potential security risk if configured incorrectly.

---

# 3. Scheduled Task Structure

Windows organizes scheduled tasks inside the:

## Task Scheduler Library

The Task Scheduler Library is not the same as a normal file system.

A task is identified by:

```text
Task Path + Task Name
```

Example:

```text
\Microsoft\Windows\Update\UpdateTask
```

Two tasks can have the same name if they are located in different folders.

Example:

Allowed:

```text
\Folder1\Backup
\Folder2\Backup
```

Not allowed:

```text
\Folder1\Backup
\Folder1\Backup
```

---

# 4. Important Scheduled Task Properties

A scheduled task contains several important components.

---

# 4.1 Triggers

A **trigger** defines when the task starts.

Examples:

### Time-based Trigger

```text
Every day at 12:00 PM
```

---

### Logon Trigger

```text
When a user logs in
```

---

### Event Trigger

```text
When a specific Windows event occurs
```

---

# 4.2 Actions

An **action** defines what the task executes.

Example:

```text
Program:

notepad.exe
```

or:

```text
powershell.exe -File script.ps1
```

The action contains:

* Executable path
* Arguments
* Working directory

---

# 4.3 Conditions and Settings

Conditions control additional requirements.

Examples:

* Only run when connected to a network
* Only run when connected to AC power
* Allow manual execution

---

# 4.4 Security Settings (Principals)

The **principal** defines:

* Which user runs the task
* What privilege level it uses

Examples:

Normal user:

```text
User: John
```

High privilege:

```text
NT AUTHORITY\SYSTEM
```

---

## Why is This Important?

A task running as:

```text
NT AUTHORITY\SYSTEM
```

has the highest privileges on Windows.

If a low-privileged user can modify what the task executes, they may achieve privilege escalation.

---

# 5. Enumerating Scheduled Tasks

Security professionals often enumerate scheduled tasks to understand system behavior.

---

# Using PowerShell

List available tasks:

```powershell
Get-ScheduledTask
```

Example output:

```text
TaskName          State
--------          -----
BackupTask        Ready
UpdateTask        Running
```

---

# Using Command Prompt

Windows also provides:

```cmd
schtasks
```

Example:

```cmd
schtasks /query
```

---

# Filtering Tasks

You can filter tasks by folder.

Example:

```powershell
Get-ScheduledTask -TaskPath "\Microsoft\Windows\Shell\"
```

This displays only tasks inside that location.

---

# Viewing Detailed Information

To view full details:

```powershell
Get-ScheduledTask -TaskName "TaskName" | Format-List
```

or:

```powershell
Get-ScheduledTask -TaskName "TaskName" | fl
```

This displays:

* Triggers
* Actions
* User account
* Settings

---

# Finding the Executed Program

The action property shows what the task runs.

Example:

```powershell
(Get-ScheduledTask -TaskName "Backup").Actions
```

Output:

```text
Execute:

powershell.exe

Arguments:

-backup.ps1
```

---

# Exporting Task Configuration

Scheduled tasks can be exported as XML.

Command:

```powershell
Export-ScheduledTask -TaskName "TaskName"
```

The XML file contains:

* Triggers
* Actions
* User account
* Permissions
* Settings

This is useful for:

* Documentation
* Automation
* Security analysis

---

# 6. Creating Scheduled Tasks

A task requires:

1. An action
2. A trigger

Example:

Create a task that opens Notepad when a user logs in.

---

## Define Action

```powershell
New-ScheduledTaskAction -Execute "notepad.exe"
```

---

## Define Trigger

```powershell
New-ScheduledTaskTrigger -AtLogOn
```

---

## Register Task

The task is then added to Windows Task Scheduler.

After registration:

```text
Task Status:

Ready
```

---

# 7. Triggering a Task

If the trigger is:

```text
At user logon
```

Logging out and logging back in will trigger it.

Flow:

```text
User Login

    ↓

Scheduled Task Trigger

    ↓

Launch Application
```

---

# 8. Removing Scheduled Tasks

To delete a task:

```powershell
Unregister-ScheduledTask
```

Example:

```powershell
Unregister-ScheduledTask -TaskName "BackupTask"
```

This removes the task from Task Scheduler.

---

# 9. Scheduled Task Privilege Escalation

Scheduled tasks can become a privilege escalation vulnerability when they are configured incorrectly.

---

# Vulnerable Scenario

Imagine a scheduled task:

```text
Task:

Backup Script

Runs As:

NT AUTHORITY\SYSTEM

Runs Every:

1 minute
```

The task executes:

```text
C:\Scripts\backup.ps1
```

---

## The Problem

If a normal user can modify:

```text
C:\Scripts\backup.ps1
```

they can replace it with a malicious script.

Example:

Before:

```powershell
backup.ps1
```

After:

```powershell
malicious.ps1
```

---

## When the Task Runs

Windows executes:

```text
Scheduled Task

        ↓

SYSTEM Account

        ↓

Malicious Script

        ↓

SYSTEM Privileges
```

The attacker gains a high-privilege shell.

---

# 10. Why Exploitation is Difficult

Although the concept is simple, finding vulnerable tasks in real systems can be difficult.

Reasons:

* Low-privileged users cannot view all tasks.
* SYSTEM-level tasks may be hidden.
* Access permissions may prevent enumeration.

Example:

A normal user may run:

```powershell
Get-ScheduledTask
```

but only see tasks they are allowed to access.

---

# 11. Practical Enumeration Advice

During penetration tests or certifications such as OSCP, manually checking scheduled tasks may not always work.

Instead, look for clues:

---

## Check Modified Files

Look for:

* Scripts that change regularly
* Executables running from unusual locations
* Writable folders

---

## Check PowerShell History

Previous commands may reveal:

* Scheduled task creation
* Script paths
* Administrative actions

Example:

```powershell
Get-PSReadLineOption
```

---

# Detection and Prevention

## Security Recommendations

To prevent scheduled task abuse:

* Use strong file permissions.
* Prevent normal users from modifying scheduled task files.
* Avoid running tasks as SYSTEM unless necessary.
* Regularly audit scheduled tasks.
* Monitor unusual task creation.

---


