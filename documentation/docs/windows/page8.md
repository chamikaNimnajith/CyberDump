

# Unquoted Service Path Vulnerability

## Overview

An **Unquoted Service Path Vulnerability** is a Windows privilege escalation vulnerability that occurs when a service's executable path:

* Contains **spaces**
* Is **not enclosed in quotation marks (`"`)**

When these two conditions are met, Windows may execute an unintended executable before the legitimate service binary.

---

## Why Does This Happen?

Windows services store the location of their executable in the service configuration.

For example:

```text
C:\Program Files\Company App\service.exe
```

Because the path contains spaces and is not enclosed in quotes, Windows cannot immediately determine which part is the executable name.

To locate the executable, Windows follows a predefined search order.

---

## Checking a Service Configuration

Use the following command to view a service's executable path:

```cmd
sc qc <service_name>
```

Example:

```cmd
sc qc VulnService
```

Look for the **BINARY_PATH_NAME** field.

### Safe Configuration

```text
"C:\Program Files\Company App\service.exe"
```

### Vulnerable Configuration

```text
C:\Program Files\Company App\service.exe
```

---

# How Windows Parses an Unquoted Path

When Windows encounters an unquoted path containing spaces, it searches for an executable by stopping at each space and appending **`.exe`**.

It continues until it reaches the intended executable.

## Example 1

Service path:

```text
C:\Program Files\Company App\service.exe
```

Windows searches in the following order:

```text
1. C:\Program.exe
2. C:\Program Files.exe
3. C:\Program Files\Company.exe
4. C:\Program Files\Company App\service.exe
```

The first executable that exists is executed.

---

## Example 2

Service path:

```text
C:\Users\User\Downloads\example directory\another directory\simple_service.exe
```

Windows checks:

```text
1. C:\Users\User\Downloads\example.exe
2. C:\Users\User\Downloads\example directory\another.exe
3. C:\Users\User\Downloads\example directory\another directory\simple_service.exe
```

---

## Example 3

Service path:

```text
C:\Program\cool company\cool banner.exe
```

Windows searches:

```text
1. C:\Program\cool.exe
2. C:\Program\cool company\cool.exe
3. C:\Program\cool company\cool banner.exe
```

---

# Exploiting the Vulnerability

An attacker can exploit this behavior if they have **write permission** to one of the directories Windows searches.

Unlike other service-related privilege escalation techniques, the attacker **does not need** permission to:

* Modify the service configuration
* Change the service binary
* Reconfigure the service

Instead, the attacker only needs to place a malicious executable in one of the searched locations.

---

## Example

Suppose Windows searches for:

```text
C:\Program.exe
```

If an attacker can create this file, Windows will execute it before attempting to launch the legitimate service.

Since services often run as **LocalSystem**, the malicious executable will inherit those privileges, potentially leading to **SYSTEM-level code execution**.

---

# Demonstration Workflow

The following steps demonstrate how the vulnerability can be exploited in a lab environment.

## Step 1 – Create the Vulnerable Service

The service executable is placed inside a directory containing spaces.

```text
Downloads\
    example directory\
        another directory\
            simple_service.exe
```

---

## Step 2 – Generate a Payload

Generate a payload using **msfvenom**.

Example:

```bash
msfvenom ...
```

Transfer the payload to the target machine using a suitable method (e.g., Netcat).

---

## Step 3 – Place the Payload

Rename the payload to match the filename Windows searches for.

Example:

```text
example.exe
```

Place it inside:

```text
Downloads\
```

---

## Step 4 – Start the Service

```cmd
sc start simple_service
```

Windows searches for:

```text
Downloads\example.exe
```

Since it exists, Windows executes the malicious executable instead of the legitimate service.

---

## Step 5 – Verify the Search Order

Remove:

```text
example.exe
```

Place the payload inside:

```text
Downloads\example directory\
```

Rename it to:

```text
another.exe
```

Start the service again.

Windows now executes **another.exe**, demonstrating that **any writable directory in the search chain can be used to hijack the service**.

---

## Filename Matters

Windows searches for exact filenames.

Example:

✔ Correct

```text
another.exe
```

✘ Incorrect

```text
another_payload.exe
```

If the filename does not exactly match what Windows expects, the exploit will fail.

---

# Detecting Unquoted Service Paths

## Using WinPEAS

WinPEAS automatically scans Windows services for common privilege escalation opportunities.

Run:

```cmd
winpeas.exe quiet servicesinfo
```

or

```cmd
winpeasx64.exe
```

If an unquoted service path is detected, WinPEAS reports something similar to:

```text
No quotes and space detected
```

This indicates that the service may be vulnerable.

---

# Remediation

The fix is straightforward: **enclose the service executable path in quotation marks.**

## Before

```text
C:\Program Files\Company App\service.exe
```

## After

```text
"C:\Program Files\Company App\service.exe"
```

Quoting the path tells Windows to treat the entire string as the executable path, preventing it from searching intermediate locations.

---

## Method 1 – Registry Editor

Navigate to:

```text
HKEY_LOCAL_MACHINE
 └── SYSTEM
     └── CurrentControlSet
         └── Services
             └── <service_name>
```

Edit the **ImagePath** value and add quotation marks around the executable path.

---

## Method 2 – Command Line

Update the service configuration using:

```cmd
sc config simple_service binpath= "\"C:\Users\User\Downloads\example directory\another directory\simple_service.exe\""
```

Verify the change:

```cmd
sc qc simple_service
```

If the path is displayed with quotation marks, the vulnerability has been remediated.

---

