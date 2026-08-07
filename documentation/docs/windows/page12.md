# Windows Credential Exposure and Sensitive Data Locations

## Overview

Windows stores and manages a large amount of sensitive information such as:

* User commands
* Password hashes
* System configuration data
* Application credentials
* Memory contents

Attackers and security professionals often examine these locations during **privilege escalation**, **forensics**, and **incident response** investigations.

---

# 1. PowerShell History Logs and Transcripts

## What is PowerShell History?

PowerShell keeps a record of commands executed by users.

This feature is useful because users can view and reuse previously executed commands.

However, it can also become a security risk if users accidentally store sensitive information in commands.

---

# Viewing Current PowerShell Session History

The command:

```powershell
Get-History
```

shows commands executed during the current PowerShell session.

Example:

```text
 id="a3pl8f"
Command
-------
whoami
ipconfig
Get-Service
```

However:

* It only shows commands.
* It does **not** show command output.
* It is temporary and disappears when the session closes.

---

# PowerShell History File

PowerShell uses the **PSReadLine** module to save command history permanently.

To find the location of the history file:

```powershell
Get-PSReadLineOption | Select-Object -Property HistorySavePath
```

---

## Windows Location

The history file is usually stored at:

```text
C:\Users\<username>\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt
```

Example:

```text
C:\Users\John\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt
```

---

## Linux Location

On Linux systems running PowerShell, the history file is stored inside the user's home directory.

Example:

```text
/home/user/.local/share/powershell/PSReadLine/ConsoleHost_history.txt
```

---

# Security Risk of PowerShell History

The history file stores commands as **plain text**.

If a user enters sensitive information directly into a command:

Example:

```powershell
mysql -u admin -p Password123
```

The password may be saved in:

```text
ConsoleHost_history.txt
```

An attacker who gains access to the user account can read this file and steal:

* Passwords
* API keys
* Database credentials
* VPN keys
* Tokens

---

# PowerShell Transcripts

PowerShell also supports **transcription**.

Unlike normal history:

| Feature    | Stores            |
| ---------- | ----------------- |
| History    | Commands only     |
| Transcript | Commands + Output |

A transcript records everything that happens during a PowerShell session.

Example:

```powershell
Start-Transcript -Path C:\temp\session.txt
```

After finishing:

```powershell
Stop-Transcript
```

---

# Environment Variables

Another important location to check is environment variables.

View them using:

```powershell
dir env:
```

Applications sometimes store:

* API keys
* Connection strings
* Authentication tokens
* Configuration values

Example:

```text
DATABASE_PASSWORD=password123
API_KEY=abc123
```

If exposed, these values can allow unauthorized access.

---

# Best Practice

Applications should avoid accepting passwords through command-line arguments.

Bad example:

```bash
application.exe --password Secret123
```

The password may appear in:

* PowerShell history
* Process lists
* Logs

Better approach:

* Ask users for credentials interactively.
* Use secure input methods.
* Store secrets using secure credential managers.

---

# 2. Security Accounts Manager (SAM) and SYSTEM Files

## What is the SAM Database?

The **Security Accounts Manager (SAM)** is a Windows database that stores information about local user accounts.

It contains:

* User account names
* Security Identifiers (SIDs)
* Password hashes (NTLM hashes)
* Group membership information

---

## SAM File Location

The SAM database is stored at:

```text
C:\Windows\System32\config\SAM
```

---

## Why Can't Users Normally Access SAM?

The SAM file is protected by Windows.

It is locked by the operating system to prevent unauthorized copying.

The **Local Security Authority Subsystem Service (LSASS)** manages authentication and protects sensitive credential information.

Process:

```text
User Login
    │
    ▼
lsass.exe
    │
    ▼
SAM Database
```

---

# Extracting Password Hashes

There are two common approaches.

---

# Method 1 – LSASS Memory Dumping

## What is LSASS?

`lsass.exe` is a Windows process responsible for:

* Authentication
* Security policies
* Credential management

When users log in, LSASS may store credential information in memory.

---

## Using Mimikatz

Attackers with administrator privileges can target LSASS memory.

Tools such as **Mimikatz** can extract:

* NTLM hashes
* Cached credentials
* Authentication tokens

The extracted hashes can potentially be used for:

* Password cracking
* Pass-the-Hash attacks
* Lateral movement

---

## Security Note

Credential dumping tools are commonly detected by antivirus solutions because they are frequently used by attackers.

---

# Method 2 – Registry Hive Export

Another method uses Windows registry backups.

This is possible when a user has special privileges.

---

## Backup Operators Group

Members of the **Backup Operators** group have:

* `SeBackupPrivilege`
* `SeRestorePrivilege`

These privileges allow them to bypass normal file permissions for backup purposes.

---

## Exporting Registry Hives

The following command can create copies of registry hives:

```cmd
reg save
```

Example:

```cmd
reg save HKLM\SAM sam.hive
```

```cmd
reg save HKLM\SYSTEM system.hive
```

The extracted files:

```text
sam.hive
system.hive
```

can then be moved to another machine for offline analysis.

---

# Extracting Hashes Offline

The attacker can analyze:

```text
SAM hive
+
SYSTEM hive
```

to recover local NTLM password hashes.

This avoids accessing the locked SAM file directly.

---

# 3. Windows Registry Hives

## What is the Windows Registry?

The Windows Registry is a hierarchical database that stores system and application settings.

It works like an upside-down tree:

```text
Registry
   │
   ├── Keys
   │      │
   │      └── Subkeys
   │
   └── Values
```

---

# Important Registry Hives

| Hive     | Purpose                           |
| -------- | --------------------------------- |
| **HKLM** | System-wide configuration         |
| **HKCU** | Current user's settings           |
| **HKCR** | File associations and COM objects |
| **HKU**  | All user profiles                 |
| **HKCC** | Hardware configuration            |

---

# Registry Hive Files

Registry hives are stored as physical files.

Common locations:

```text
C:\Windows\System32\Config
```

Examples:

```text
SAM
SYSTEM
SECURITY
SOFTWARE
```

User-specific hives:

```text
C:\Users\<username>\NTUSER.DAT
```

---

# Security Importance of Registry Hives

The Registry contains important information about:

* Installed software
* Startup programs
* User settings
* Security configurations

Malware often modifies registry keys to:

* Maintain persistence
* Disable security tools
* Execute automatically

Security analysts investigate registry changes as:

**Indicators of Compromise (IoCs)**

---

# Parsing Registry Hives

Security researchers can analyze offline registry files.

Example Python package:

```bash
pip install regipy
```

`regipy` allows analysts to:

* Read registry hive files
* Search keys
* Extract configuration information

Example:

```python
from regipy.registry import RegistryHive

hive = RegistryHive("SYSTEM")
```

---

# 4. Configuration Files, Paging, and Hibernation Files

## Application Configuration Files

Applications commonly store configuration data inside:

```text
AppData\Roaming
```

and:

```text
AppData\Local
```

Example:

```text
C:\Users\User\AppData\Roaming\Application
```

These folders may contain:

* Configuration files
* Saved sessions
* Tokens
* Credentials

---

## Security Risk

Some applications store sensitive data in plain text.

Examples:

* Browser data
* FTP credentials
* Database passwords
* API keys

Security assessments should always review application configuration locations.

---

# Paging File

## What is the Paging File?

The paging file (`pagefile.sys`) is used by Windows when RAM becomes full.

Windows moves inactive memory data from RAM to disk.

Location:

```text
C:\pagefile.sys
```

---

## Security Risk

RAM may temporarily contain:

* Passwords
* Encryption keys
* User data

When memory is moved to the paging file, sensitive information may be written to disk.

---

# Hibernation File

## What is Hibernation?

During hibernation, Windows saves the contents of RAM to:

```text
hiberfil.sys
```

This allows the computer to resume quickly.

---

## Security Risk

Because RAM contents are stored on disk, the hibernation file may contain:

* Credentials
* Application data
* Encryption keys
* User activity information

Forensic investigators can analyze these files to recover information about system activity.

---
