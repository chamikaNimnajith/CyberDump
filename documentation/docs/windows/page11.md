# AlwaysInstallElevated 

## Overview

**AlwaysInstallElevated** is a Windows Group Policy setting that allows **MSI (Microsoft Installer)** packages to be installed with **SYSTEM privileges**.

If this feature is **incorrectly enabled**, a normal user can install a **malicious MSI package** and execute code with the highest privileges on the computer.

This is a common **Windows privilege escalation** vulnerability.

---

# 1. What is AlwaysInstallElevated?

Normally, installing software requires **administrator privileges** because installers often:

* Copy files into protected system folders.
* Modify the Windows Registry.
* Install services.
* Change system-wide settings.

If the **AlwaysInstallElevated** policy is enabled, Windows Installer automatically runs MSI packages as:

```text
NT AUTHORITY\SYSTEM
```

This means **even a standard user** can execute an MSI installer with **SYSTEM privileges**.

---

## Why is this Dangerous?

If users can install **any MSI package**, they can also install a **malicious MSI**.

Since Windows Installer runs it as **SYSTEM**, the attacker's code also executes as **SYSTEM**.

This allows the attacker to gain complete control of the machine.

---

# 2. Checking the Configuration

For this vulnerability to exist:

✅ **Computer Configuration** must be enabled.

AND

✅ **User Configuration** must also be enabled.

If only one setting is enabled, the vulnerability **does not exist**.

---

## Method 1 – Using Group Policy

Open the Local Group Policy Editor:

```text
gpedit.msc
```

Check the following policy:

### Computer Configuration

```text
Computer Configuration
    └── Administrative Templates
        └── Windows Components
            └── Windows Installer
                └── Always install with elevated privileges
```

---

### User Configuration

```text
User Configuration
    └── Administrative Templates
        └── Windows Components
            └── Windows Installer
                └── Always install with elevated privileges
```

If **both** are **Enabled**, the system is vulnerable.

---

# 3. Checking Using the Registry

The same settings can be checked from the Windows Registry.

### Computer-wide setting

```text
HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer
```

---

### User setting

```text
HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer
```

Look for the value:

```text
AlwaysInstallElevated
```

---

## Registry Values

| Value | Meaning         |
| ----- | --------------- |
| **0** | Disabled (Safe) |
| **1** | Enabled         |

For exploitation, **both registry values must be 1**.

---

# 4. Normal vs Vulnerable Behavior

## Normal Configuration

If the policy is disabled:

```text
AlwaysInstallElevated = 0
```

When a user runs an MSI installer:

* Windows displays a **UAC prompt**.
* Administrator credentials are required.
* Standard users cannot install privileged software.

---

## Vulnerable Configuration

If both policies are enabled:

```text
AlwaysInstallElevated = 1
```

When a user runs an MSI installer:

* No administrator credentials are required.
* Windows Installer executes the MSI as **SYSTEM**.
* The installer gains the highest privilege level on the machine.

---

# 5. How the Vulnerability is Exploited

An attacker creates a **malicious MSI installer** containing a custom payload.

The attacker simply runs the installer.

Because **AlwaysInstallElevated** is enabled, Windows Installer launches it with **SYSTEM privileges**.

As a result, the attacker's code executes as:

```text
NT AUTHORITY\SYSTEM
```

This gives the attacker full control over the system.

---

# 6. Running the Installer

MSI packages are executed using the **msiexec** utility.

Example:

```cmd
msiexec /i malicious.msi
```

Attackers often use silent installation options to avoid displaying installer windows.

Common options include:

```text
/quiet
/q
/i
```

These options allow the installer to run quietly in the background.

---

# 7. Creating a Custom MSI

A custom MSI package can be created using the **WiX Toolset**.

The video demonstrates **WiX Toolset 3.14.1**.

---

## Step 1 – Create a .wxs File

A **.wxs** file is an XML document describing:

* Files to install
* Installation actions
* Custom commands to execute

---

## Step 2 – Compile the Project

Use **candle.exe** to compile the `.wxs` file.

Example:

```text
.wxs
    │
    ▼
candle.exe
    │
    ▼
.wixobj
```

---

## Step 3 – Build the MSI

Use **light.exe** to convert the `.wixobj` file into an MSI package.

```text
.wixobj
    │
    ▼
light.exe
    │
    ▼
malicious.msi
```

---

# 8. Important WiX Setting

When defining a custom action in the `.wxs` file, the **Impersonate** attribute is very important.

### Impersonate="yes"

```text
Impersonate="yes"
```

The custom action runs as the **current user**.

No privilege escalation occurs.

---

### Impersonate="no"

```text
Impersonate="no"
```

The custom action runs as the **SYSTEM** account.

This allows the malicious command to execute with the highest privileges.

---

# Exploitation Flow

```text
AlwaysInstallElevated Enabled
            │
            ▼
User runs malicious.msi
            │
            ▼
Windows Installer (msiexec)
            │
            ▼
Installer executes as SYSTEM
            │
            ▼
Attacker gains SYSTEM privileges
```

---

# Detection Tips

Check whether **AlwaysInstallElevated** is enabled:

* Using **Group Policy (`gpedit.msc`)**
* Checking the Registry (`HKLM` and `HKCU`)

The system is only vulnerable if **both** settings are enabled.

---

# Prevention

To prevent this privilege escalation:

* Keep **AlwaysInstallElevated** disabled.
* Ensure both registry values are set to **0**.
* Restrict users from installing untrusted software.
* Regularly audit Group Policy settings.

---

