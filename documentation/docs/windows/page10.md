# User Account Control (UAC) and Mandatory Integrity Control (MIC)

## Overview

Windows uses several security mechanisms to prevent unauthorized users or applications from gaining administrative privileges. Two of the most important are:

* **User Account Control (UAC)**
* **Mandatory Integrity Control (MIC)**

Although they work together, they have different responsibilities.

---

# 1. What is User Account Control (UAC)?

**User Account Control (UAC)** is a Windows security feature that prevents programs from automatically running with administrator privileges.

When a program needs administrator rights, Windows displays a **UAC prompt** asking the user for permission (or an administrator password, depending on the configuration).

This helps prevent malware or unauthorized applications from silently gaining full control of the system.

### Example

Suppose you open **Command Prompt** normally.

It runs with **standard user privileges**.

If you select **Run as administrator**, Windows displays a UAC prompt.

Only after you approve the prompt does the program run with administrator privileges.

---

# 2. What is Mandatory Integrity Control (MIC)?

**Mandatory Integrity Control (MIC)** is a Windows access control mechanism that assigns an **Integrity Level** to every process and object.

The integrity level determines what a process is allowed to modify.

A process with a lower integrity level **cannot modify** objects that have a higher integrity level.

---

## Windows Integrity Levels

Windows defines four main integrity levels.

| Integrity Level | Description                                                                     |
| --------------- | ------------------------------------------------------------------------------- |
| **Low**         | Very limited privileges (e.g., sandboxed applications, browser protected mode). |
| **Medium**      | Standard user applications. Most programs run at this level.                    |
| **High**        | Administrator applications running with elevated privileges.                    |
| **System**      | Highest privilege level used by Windows services and the operating system.      |

---

## Example

Suppose:

* Notepad runs as **Medium Integrity**.
* Registry Editor runs as **High Integrity**.

A Medium Integrity process **cannot modify** High Integrity objects.

This prevents normal applications from changing administrative settings.

---

# 3. How UAC and MIC Work Together

MIC defines the **security boundaries**.

UAC controls **when a process is allowed to cross those boundaries**.

For example:

```text
Medium Integrity
        │
        │  User clicks "Run as Administrator"
        ▼
      UAC Prompt
        │
        ▼
High Integrity
```

Without UAC approval, a Medium Integrity process cannot become High Integrity.

---

## Important Note

Even if your Windows account belongs to the **Administrators** group, programs do **not** automatically run as administrators.

For example:

* PowerShell
* Command Prompt
* File Explorer

normally start with **Medium Integrity**.

Only after UAC approval do they run with **High Integrity**.

---

# 4. UAC Configuration Levels

Windows allows administrators to configure how UAC behaves.

Different settings provide different levels of security.

---

## Level 0 – No Prompt

At this level, UAC is effectively disabled.

Applications can elevate to administrator privileges **without asking the user**.

### Security Impact

* Very insecure.
* Malware can gain administrator privileges silently.

---

## Level 1 – Prompt for Credentials (Secure Desktop)

Windows switches to the **Secure Desktop**.

The user must enter an administrator password.

### Characteristics

* Requires administrator credentials.
* Secure Desktop prevents other applications from interacting with the prompt.
* Provides strong protection.

---

## Level 2 – Prompt for Consent (Secure Desktop)

Windows displays the Secure Desktop.

Instead of entering a password, the user simply clicks **Yes**.

### Characteristics

* Secure Desktop is still used.
* Safer than normal desktop prompts.
* Easier for administrators.

---

## Level 3 – Prompt for Credentials (Normal Desktop)

The password prompt appears on the normal desktop.

Other applications continue running.

### Characteristics

* Requires credentials.
* Less secure because the desktop is not isolated.

---

## Level 4 – Prompt for Consent (Normal Desktop)

Windows asks the user for confirmation using the normal desktop.

### Characteristics

* User clicks **Yes**.
* Easier to use.
* Less secure than Secure Desktop.

---

## Level 5 – Prompt Only for Non-Windows Programs

This is the default setting on many Windows systems.

Microsoft-signed Windows applications are **automatically trusted**.

Only third-party applications trigger a UAC prompt.

### Characteristics

* Trusted Windows programs auto-elevate.
* Non-Microsoft programs require user approval.

---

# 5. UAC Bypass Techniques

A **UAC bypass** is a technique that allows a program to obtain elevated privileges without displaying the expected UAC prompt.

Different bypasses work depending on the UAC configuration.

---

# Bypass 1 – Level 0 (UAC Disabled)

If UAC is disabled, bypassing it is trivial.

A program can simply request administrator privileges.

Example:

```powershell
Start-Process powershell -Verb RunAs
```

Since no prompt is displayed, PowerShell starts with **High Integrity**.

---

# Bypass 2 – fodhelper.exe (Level 5)

One of the most common UAC bypass techniques uses **fodhelper.exe**.

## What is fodhelper.exe?

`fodhelper.exe` is a legitimate Microsoft application used to manage **Features on Demand**.

Because it is Microsoft-signed, Windows allows it to **auto-elevate** without displaying a UAC prompt on **Level 5** systems.

---

## How the Attack Works

The attacker creates specific registry keys under:

```text
HKCU\Software\Classes\ms-settings\Shell\Open\command
```

The registry is modified to point to a malicious command.

When **fodhelper.exe** starts:

1. Windows automatically elevates it.
2. It reads the attacker-controlled registry key.
3. The attacker's command executes with **High Integrity**.

---

## Why Does It Work?

* `fodhelper.exe` is trusted by Windows.
* It is configured with:

```text
autoElevate = true
```

Therefore, Windows does not display a UAC prompt on Level 5 systems.

---

## Limitation

This technique only works when UAC is configured to trust Microsoft binaries.

It **does not bypass** Levels **1–4**, where Windows always asks the user for credentials or consent.

---

# Bypass 3 – AlwaysInstallElevated

Another privilege escalation technique targets the **AlwaysInstallElevated** policy.

## What is AlwaysInstallElevated?

Windows Installer (MSI) packages normally require administrator privileges.

However, if the **AlwaysInstallElevated** policy is enabled, any MSI package is installed with **SYSTEM** privileges.

---

## Attack Process

1. Create a malicious MSI installer.
2. Execute it using:

```cmd
msiexec /i malicious.msi
```

If the policy is enabled:

* The MSI runs as **NT AUTHORITY\SYSTEM**.
* The attacker gains the highest privilege level on the machine.

---

## Why is it Dangerous?

SYSTEM has more privileges than Administrator.

This completely bypasses the normal UAC prompts.

---

# DLL Hijacking and UAC Bypass

Many UAC bypass techniques rely on **DLL Hijacking**.

Instead of directly attacking UAC, the attacker tricks a trusted Windows application into loading a malicious DLL.

Since the trusted application already runs with elevated privileges, the malicious DLL also executes with those privileges.

---

# Summary of UAC Bypass Techniques

| Technique                 | Works Against                           | Description                                                                                                   |
| ------------------------- | --------------------------------------- | ------------------------------------------------------------------------------------------------------------- |
| **Level 0**               | UAC Disabled                            | No prompt is displayed, allowing immediate elevation.                                                         |
| **fodhelper.exe**         | Level 5                                 | Uses Microsoft's auto-elevating executable and registry manipulation to execute commands with High Integrity. |
| **AlwaysInstallElevated** | Levels 1–4 (if policy is misconfigured) | Executes a malicious MSI installer as **SYSTEM**.                                                             |
| **DLL Hijacking**         | Various scenarios                       | Tricks an elevated application into loading a malicious DLL.                                                  |

---

