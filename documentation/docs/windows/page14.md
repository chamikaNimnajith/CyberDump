# Windows Credential Manager and Credential Vault 

## Overview

Windows provides built-in mechanisms to store and manage user credentials securely.

These features allow users and applications to automatically access saved:

* Usernames
* Passwords
* Certificates
* Authentication information

The two main components are:

1. **Windows Credential Manager**
2. **Windows Vault**

During security assessments, these locations are important because saved credentials can sometimes allow attackers to move between user accounts or gain higher privileges.

---

# 1. Windows Credential Manager

## What is Credential Manager?

**Windows Credential Manager** is a Windows feature that stores user credentials so users do not need to enter passwords repeatedly.

Examples of stored credentials:

* Network shares
* Remote Desktop connections
* Websites
* Applications
* Internal company resources

---

## Example

A user connects to a remote computer:

```text
Remote Server:
192.168.1.50

Username:
admin

Password:
********
```

Windows can save these credentials.

The next time the user connects, Windows automatically retrieves them.

---

# How Credential Manager Protects Data

Credentials stored in Credential Manager are encrypted using:

## DPAPI (Data Protection API)

DPAPI is a Windows encryption system used to protect sensitive user data.

The stored credentials are encrypted on disk.

Example:

```text
Password

    ↓

DPAPI Encryption

    ↓

Encrypted Credential File
```

---

## Security Risk

Although credentials are encrypted, they can become dangerous if an attacker gains access to:

* An active user session
* A compromised account
* A machine where sensitive credentials are stored

If the attacker is running as that user, Windows may allow access to the saved credentials.

---

# 2. Using Credential Manager with cmdkey

Windows provides a command-line tool called:

```cmd
cmdkey
```

It allows users to manage stored credentials.

---

# Listing Saved Credentials

To view stored credentials:

```cmd
cmdkey /list
```

Example output:

```text
Currently stored credentials:

Target:
Domain:target=server01

User:
admin
```

---

# Security Enumeration

During a security assessment, checking Credential Manager is an important step.

Example scenario:

An attacker gains access to a machine as:

```text
User:
Leonardo
```

They run:

```cmd
cmdkey /list
```

The output shows:

```text
Saved credential:

User:
quick ammo
```

This indicates another user's credentials may exist on the system.

---

# Using Saved Credentials for Pivoting

Windows provides the:

```cmd
runas
```

command.

It allows a user to execute a program as another account.

---

## /savecred Option

Example:

```cmd
runas /savecred /user:username cmd.exe
```

The first time:

1. Windows asks for the password.
2. The credentials are saved.

Later executions:

1. Windows retrieves the stored credentials.
2. The application starts without asking for the password.

---

## Example Flow

```text
Saved Credentials

        ↓

runas /savecred

        ↓

Windows retrieves password

        ↓

Command Prompt runs as another user
```

---

# Important Limitation

Credential Manager does **not store passwords in plain text**.

An attacker cannot simply run:

```text
cmdkey /list
```

and see:

```text
Password:
Password123
```

Instead, Windows stores encrypted credentials.

---

# Deleting Stored Credentials

A user can remove saved credentials using:

```cmd
cmdkey /delete:<Target>
```

Example:

```cmd
cmdkey /delete:server01
```

This removes the stored credential.

---

# Adding Credentials

Credentials can also be added manually:

```cmd
cmdkey /add:<Target> /user:<Username> /pass:<Password>
```

Example:

```cmd
cmdkey /add:server01 /user:admin /pass:Password123
```

---

# 3. Windows Vault

## What is Windows Vault?

The **Windows Vault** is the underlying storage system used by Credential Manager.

Credential Manager provides a simple user interface.

The Windows Vault provides the deeper storage mechanism.

Relationship:

```text
Windows Vault
       │
       ▼
Credential Manager
       │
       ▼
Saved Credentials
```

---

# Credential Manager vs Windows Vault

| Feature        | Credential Manager       | Windows Vault                   |
| -------------- | ------------------------ | ------------------------------- |
| User Interface | Simple                   | Advanced                        |
| Purpose        | Manage saved credentials | Store encrypted credential data |
| Tool           | cmdkey                   | vaultcmd                        |
| Storage        | Uses Vault internally    | Direct vault access             |

---

# 4. Using vaultcmd

Because the Windows Vault is separate from Credential Manager, it uses a different command:

```cmd
vaultcmd
```

---

# Listing Available Vaults

Command:

```cmd
vaultcmd /list
```

Example output:

```text
Vault:
Web Credentials

Vault:
Windows Credentials
```

Each vault has:

* A unique identifier (GUID)
* A storage location

---

# Vault Storage

Vault data is stored in encrypted files.

Example:

```text
policy.vpol
```

These files are protected and cannot be opened directly as normal text files.

---

# Listing Credentials Inside a Vault

Command:

```cmd
vaultcmd /listcreds "<Vault Name>" /all
```

Example:

```cmd
vaultcmd /listcreds "Windows Credentials" /all
```

This displays information stored inside the selected vault.

---

# Dumping Vault Information

Security tools can analyze Windows Vault data.

For example:

* Mimikatz
* Other credential recovery tools

These tools attempt to extract:

* Vault keys
* Encryption material
* Stored credentials

Security tools may detect these actions because credential extraction is commonly used by malware.

---

# 5. Security Assessment Workflow

During a Windows security assessment, credential discovery is an important step.

A common enumeration process:

---

## Step 1: Check Stored Credentials

Run:

```cmd
cmdkey /list
```

Look for:

* Other usernames
* Remote systems
* Saved authentication information

---

## Step 2: Check Vault Information

Run:

```cmd
vaultcmd /list
```

and:

```cmd
vaultcmd /listcreds "<Vault Name>" /all
```

---

## Step 3: Identify Possible Privilege Escalation Paths

Saved credentials may allow:

* Access to another user account
* Lateral movement
* Privilege escalation

---

# Example Attack Scenario

A low-privileged user gains access:

```text
Current User:

Leonardo
```

The attacker checks:

```cmd
cmdkey /list
```

Finds:

```text
Saved Credential:

quick ammo
```

The attacker can attempt to use the stored credential:

```cmd
runas /savecred /user:"quick ammo" cmd.exe
```

A new command prompt may open under the other user's account.

---

# Important Notes

## Stored Credentials Do Not Always Mean Administrator Access

Finding another user's credentials does not automatically provide administrator privileges.

The attacker may still need:

* Additional privilege escalation techniques
* UAC bypass
* Local administrator access

---

# Prevention

To reduce credential exposure:

* Avoid saving passwords unnecessarily.
* Remove old saved credentials.
* Use strong authentication methods.
* Enable Multi-Factor Authentication (MFA).
* Limit administrative credentials stored on user machines.
* Monitor suspicious access to credential storage.

---

