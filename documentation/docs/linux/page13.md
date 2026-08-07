# Linux Kernel Exploitation and Reverse Shell Fundamentals

## What You Will Learn

In this guide, you will learn:

* The difference between binary exploitation and kernel exploitation
* How the Linux kernel interacts with user applications
* How system calls work
* How kernel vulnerabilities can lead to privilege escalation
* The concept behind the Dirty Cow vulnerability
* How reverse shells provide interactive access after gaining code execution

> **Note:** These concepts are intended for cybersecurity education, vulnerability research, and authorized penetration testing. Always practice exploitation techniques only in controlled environments.

---

# Part 1: Binary Exploitation vs Kernel Exploitation

Privilege escalation on Linux commonly happens through two major categories:

1. **User-space binary exploitation**
2. **Kernel exploitation**

Both can lead to elevated privileges, but they attack different parts of the system.

---

# Binary Exploitation (User Space)

## What is User Space?

User space is where normal applications run.

Examples:

* Web servers
* Text editors
* Database applications
* Custom programs

These programs do not directly control hardware. They communicate with the kernel through system calls.

---

## Binary Exploitation Concept

Binary exploitation focuses on finding vulnerabilities inside executable programs.

Common vulnerabilities include:

* Buffer overflows
* Format string vulnerabilities
* Use-after-free bugs
* Memory corruption issues

Example:

```text
Application Binary

        ↓

Memory Vulnerability

        ↓

Control Program Execution

        ↓

Execute Arbitrary Code
```

---

# SUID Binary Exploitation

A common privilege escalation scenario involves vulnerable **SUID binaries**.

## What is SUID?

SUID (Set User ID) allows a program to execute with the permissions of its owner.

Example:

```text
File:

/usr/bin/example


Owner:

root


SUID Enabled:

Yes
```

A normal user running this program executes it with root privileges.

---

## Exploitation Scenario

Imagine a root-owned SUID program contains a buffer overflow.

The attack flow:

```text
Normal User

      ↓

Run SUID Program

      ↓

Exploit Memory Vulnerability

      ↓

Execute Code

      ↓

Root Shell
```

Because the program already has root privileges, the attacker's code also runs as root.

---

# Kernel Exploitation

Kernel exploitation targets vulnerabilities inside the Linux kernel itself.

The kernel is the most privileged part of the operating system.

A successful kernel exploit can allow an attacker to:

* Execute code as root
* Modify system behavior
* Disable security mechanisms
* Install rootkits
* Maintain persistence

The execution flow:

```text
User Process

      ↓

Kernel Vulnerability

      ↓

Kernel Code Execution

      ↓

Full System Control
```

---

# Part 2: Understanding the Linux Kernel

## What is the Kernel?

The **kernel** is the core component of an operating system.

It acts as a bridge between:

* Hardware
* Software applications

Without the kernel, applications would need to understand every hardware device individually.

---

# Kernel Abstraction

Without a kernel:

```text
Application

↓

Hardware Specific Code

↓

CPU / Memory / Disk / Network
```

Every application would need different code for every device.

---

With a kernel:

```text
Application

↓

System Calls

↓

Kernel

↓

Hardware
```

The kernel provides a standard interface.

---

# User Space and Kernel Space

Linux separates execution into two areas.

---

## User Space

Where normal applications run.

Examples:

* Browser
* Python scripts
* Web applications

User programs have limited privileges.

---

## Kernel Space

Where the kernel operates.

The kernel has access to:

* Hardware
* Memory management
* Processes
* Devices

Kernel code runs with the highest privileges.

---

# System Calls

Applications cannot directly communicate with hardware.

Instead, they request services from the kernel using:

```text
System Calls
```

A system call is an API provided by the kernel.

Example:

A program wants to print text:

```text
Application

↓

write() System Call

↓

Kernel

↓

Display Output
```

---

# Example: Running a Simple Program

Even a basic C program uses many system calls.

Example:

```c
printf("Hello");
```

Behind the scenes:

```text
Program Starts

↓

execve()

↓

Memory Allocation

↓

mmap()

brk()

↓

Write Output

↓

write()
```

---

# Important System Calls

## execve()

Starts a new program.

Example:

```text
Run:

./program

↓

execve()
```

---

## mmap()

Maps memory regions for a process.

Used for:

* Loading libraries
* Allocating memory

---

## brk()

Changes the size of the process heap.

Used for dynamic memory allocation.

---

## write()

Writes data to a file descriptor.

Example:

```text
write(fd, data)
```

File descriptor:

```text
1 = Standard Output
```

Meaning:

```text
write()

↓

Terminal Screen
```

---

# Using strace

The Linux tool:

```bash
strace
```

allows you to monitor system calls made by a program.

Example:

```bash
strace ./program
```

Output:

```text
execve()

mmap()

brk()

write()
```

This helps understand how programs communicate with the kernel.

---

# Part 3: Kernel Exploitation Example - Dirty Cow

## Hack The Box Valentine Scenario

The demonstration uses an outdated Linux system.

The goal is to escalate privileges using a kernel vulnerability.

---

# Step 1: Kernel Enumeration

First, identify the kernel version.

Command:

```bash
uname -a
```

Example output:

```text
Linux Ubuntu

Kernel:

3.2.0
```

The kernel version is extremely important because vulnerabilities are often tied to specific versions.

---

# Step 2: Searching for Kernel Exploits

Attackers can search known vulnerabilities using:

```bash
searchsploit
```

Example:

```bash
searchsploit Linux kernel 3.2
```

This searches a local exploit database.

The search reveals:

```text
Dirty Cow

CVE-2016-5195
```

---

# What is Dirty Cow?

Dirty Cow is a Linux kernel vulnerability caused by a race condition.

It affects the kernel's:

```text
Copy-on-Write (COW)
```

memory mechanism.

---

# Understanding Copy-on-Write

Normally, multiple processes can share the same memory.

Example:

```text
Process A

      |
      |
 Shared Memory

      |
      |

Process B
```

If one process modifies the memory:

```text
Process A Writes Data
```

The kernel creates a separate copy.

This protects the other process.

```text
Before:

Shared Memory


After Write:

Process A → Copy

Process B → Original
```

This is called:

**Copy-on-Write**

---

# The Dirty Cow Race Condition

The vulnerability exists because of a timing issue.

The attacker repeatedly performs operations faster than the kernel can properly handle.

This creates a race condition:

```text
Normal:

Check Permission

↓

Copy Memory

↓

Write Data


Vulnerable:

Check Permission

↓

Race Condition

↓

Unauthorized Write
```

---

# Dirty Cow Impact

The vulnerability allows an attacker to modify files that should only be readable.

A common target is:

```text
/etc/passwd
```

Normally:

```text
/etc/passwd

Readable by users

Protected from modification
```

Dirty Cow bypasses this protection.

---

# Exploit Execution Process

The general workflow:

```text
Download Exploit

        ↓

Compile Exploit

        ↓

Run Exploit

        ↓

Modify /etc/passwd

        ↓

Create Root User

        ↓

Login as Root
```

---

# Compiling the Exploit

The exploit is compiled using:

```bash
gcc -pthread exploit.c -o exploit -lcrypt
```

Important options:

* `-pthread` → Enables threading support
* `-lcrypt` → Links password hashing library

---

# Creating a Root Account

The exploit creates a new account.

Example:

```text
Username:

fireart


UID:

0
```

In Linux:

```text
UID 0 = root
```

Therefore:

```text
fireart

↓

Root privileges
```

---

# Password Hash Technique

Modern Linux systems store passwords like:

```text
/etc/passwd

username:x:UID:GID
```

The real hash is stored in:

```text
/etc/shadow
```

However, older Unix systems allowed password hashes directly inside:

```text
/etc/passwd
```

The exploit abuses this compatibility feature.

The result:

```text
New Root User

↓

Password Hash Stored

↓

su fireart

↓

Root Access
```

---

# Part 4: Reverse Shell Fundamentals

## Why Use a Reverse Shell?

When attackers achieve Remote Code Execution (RCE), they may initially execute commands through:

* HTTP requests
* Web interfaces
* Limited command execution

This is inconvenient.

A reverse shell provides:

* Interactive command execution
* Stable access
* Ability to perform enumeration
* Easier privilege escalation

---

# Bind Shell vs Reverse Shell

## Bind Shell

In a bind shell:

1. Target opens a listening port.
2. Attacker connects to it.

Diagram:

```text
Attacker

     |
     |
     ▼

Target

Listening Port
```

---

## Problems

Bind shells are often unreliable because:

### Firewalls

Incoming connections may be blocked.

### NAT

Private machines cannot easily receive external connections.

---

# Reverse Shell

A reverse shell changes the direction.

The attacker starts a listener:

```text
Attacker

Listening

     ▲

     |

Target

Connects Out
```

The target creates the connection.

---

# Why Reverse Shells Are Preferred

## Firewall Advantage

Most networks allow:

```text
Outgoing Connections
```

but restrict:

```text
Incoming Connections
```

---

## NAT Advantage

With NAT:

```text
Internet

    |

Router

    |

Private Machine
```

The attacker cannot directly connect.

A reverse shell avoids this because the internal machine creates the connection.

---

# File Transfer

Sometimes payloads need to be transferred.

The attacker can host files using:

```bash
python3 -m http.server 1337
```

The victim can download using:

```bash
curl
```

or:

```bash
wget
```

---

# Setting Up a Listener

The attacker waits using:

```bash
nc -lvnp 4444
```

Options:

* `l` → Listen
* `v` → Verbose
* `n` → No DNS lookup
* `p` → Port

---

# How Reverse Shells Work

Every reverse shell performs three main actions.

---

## 1. Create TCP Connection

The target connects back:

```text
Victim

      TCP Socket

        ↓

Attacker
```

---

## 2. Spawn a Shell

The target starts:

```text
/bin/sh

or

/bin/bash
```

---

## 3. Redirect Input and Output

The shell streams:

* Input
* Output
* Error messages

through the network connection.

This gives the attacker interactive control.

---

# Reverse Shell Languages

Reverse shells can be created using:

* Bash
* Python
* Perl
* PHP
* Ruby

The exact syntax changes, but the underlying process remains the same.

---

# Upgrading to a TTY Shell

Basic reverse shells often lack:

* Command history
* Tab completion
* Proper terminal formatting

A better shell can be created using Python:

```bash
python3 -c "import pty; pty.spawn('/bin/bash')"
```

---

# Benefits of a TTY Shell

A TTY shell provides:

* Better interaction
* Stable terminal behavior
* Support for interactive applications

---

# Complete Attack Flow

```text
Gain RCE

   ↓

Spawn Reverse Shell

   ↓

Enumerate System

   ↓

Find Vulnerability

   ↓

Exploit Binary or Kernel

   ↓

Privilege Escalation

   ↓

Root Access
```

---

# Key Takeaways

* Binary exploitation targets vulnerable user-space applications.
* Kernel exploitation targets vulnerabilities inside the operating system kernel.
* The kernel provides hardware abstraction and exposes functionality through system calls.
* Tools like `strace` help observe kernel interactions.
* Dirty Cow exploited a race condition in Linux Copy-on-Write memory handling.
* Kernel exploits can provide complete root-level control.
* Reverse shells provide interactive access after gaining code execution.
* Reverse shells are preferred because outbound connections usually bypass firewall and NAT restrictions.
* Upgrading shells to TTY improves stability and usability.
