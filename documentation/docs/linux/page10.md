# Linux Capabilities and Privilege Escalation

## What You Will Learn

In this guide, you'll learn:

* What Linux capabilities are
* Why Linux uses capabilities instead of giving every program full root privileges
* Some of the most dangerous capabilities
* How to view and configure capabilities
* How to enumerate capabilities during a penetration test
* How misconfigured capabilities can lead to privilege escalation

> **Note:** The techniques discussed here are for cybersecurity education, penetration testing, and defensive security assessments. Only perform them on systems you own or are authorized to test.

---

# What are Linux Capabilities?

Traditionally, Linux followed a very simple permission model.

There were only **two types of processes**:

* **Root (UID = 0)** → Has almost unlimited privileges.
* **Normal User (UID ≠ 0)** → Has limited privileges.

```text
Traditional Permission Model

Root
 │
 └── Full system privileges

Normal User
 │
 └── Limited permissions
```

This model worked well, but it had one major problem.

Sometimes a program only needed **one special privilege**, but it had to run as **root** to obtain it.

This violated an important security principle.

---

# The Principle of Least Privilege

The **Principle of Least Privilege** states:

> A program should only have the permissions it absolutely needs—nothing more.

Instead of giving an application complete root privileges, Linux allows administrators to assign **specific capabilities**.

Each capability grants one particular privilege.

This reduces the damage if the application is compromised.

---

# The Capability Model

Starting with **Linux Kernel 2.2**, Linux introduced **Capabilities**.

Instead of treating privileges as "all or nothing," Linux breaks them into smaller permissions.

For example:

```text
Root Privileges

├── Network Administration
├── Load Kernel Modules
├── Change User ID
├── Debug Processes
├── Mount Filesystems
└── Many more...
```

Each of these can be granted individually.

This means a program can receive **only the capability it requires** without becoming a full root process.

---

# Example: The `ping` Command

The `ping` command needs to create **raw network sockets**.

Normally, creating raw sockets requires root privileges.

Without capabilities:

```text
ping

↓

Runs as Root

↓

Gets ALL root privileges
```

This is risky.

If someone exploits a vulnerability in `ping`, they could gain full root access.

---

With Linux capabilities:

```text
ping

↓

Only receives:

cap_net_raw

↓

Can create raw sockets

↓

Nothing else
```

Now the program only has the permission it needs.

This is much safer.

---

# Common Linux Capabilities

Linux supports many capabilities.

Some are harmless, while others are extremely powerful.

Let's look at the most important ones.

---

# `cap_net_raw`

Allows a program to create **raw network sockets**.

Commonly used by:

* `ping`
* Network monitoring tools
* Packet generators

---

# `cap_sys_admin`

This is one of the most powerful Linux capabilities.

It allows many administrative operations, including:

* Mounting filesystems
* Managing namespaces
* Various system administration tasks

It is often called the **"catch-all capability"** because it grants many powerful operations.

If a program has this capability, it may be possible to escape containers such as Docker by mounting the host filesystem.

---

# `cap_sys_ptrace`

Allows a process to:

* Debug other processes
* Inspect memory
* Modify running programs

Normally, users can only debug their own processes.

With this capability, a process may be able to interact with processes owned by other users, including root.

---

# `cap_sys_module`

Allows loading and unloading **Linux Kernel Modules (LKMs).**

Kernel modules execute inside the Linux kernel itself.

If abused, an attacker could load malicious code directly into the kernel.

---

# `cap_setuid`

Allows a program to change its **User ID (UID).**

Normally:

```text
User

↓

UID = 1000
```

With this capability, a process can change its UID.

For example:

```text
UID = 1000

↓

UID = 0

↓

Root
```

This makes `cap_setuid` one of the most dangerous capabilities.

---

# `cap_chown`

Allows changing the owner (UID) and group (GID) of files.

Normally, only privileged users can perform these operations.

---

# `cap_fowner`

Normally, users can only modify files they own.

This capability allows a program to bypass many ownership checks.

It can perform operations that would normally require file ownership.

---

# Viewing Capabilities

You can inspect the capabilities assigned to a binary using the `getcap` command.

Example:

```bash
getcap /usr/bin/ping
```

Example output:

```text
/usr/bin/ping cap_net_raw=ep
```

This shows that the `ping` binary has the `cap_net_raw` capability.

---

# Understanding `=ep`

Capabilities often end with:

```text
=ep
```

Meaning:

* **e** → Effective
* **p** → Permitted

These flags determine how the capability is applied when the program runs.

---

# Adding Capabilities

Capabilities can be added using the `setcap` command.

Example:

```bash
sudo setcap cap_net_raw+ep /path/to/program
```

This grants the program the `cap_net_raw` capability.

---

# Removing Capabilities

Capabilities can also be removed.

Example:

```bash
sudo setcap -r /path/to/program
```

or remove a specific capability:

```bash
sudo setcap cap_net_raw-ep /path/to/program
```

Once removed, the program no longer has that privilege.

---

# Enumerating Capabilities

During privilege escalation, you should always search for binaries with dangerous capabilities.

There are several ways to do this.

---

# Method 1 – Scan the Entire Filesystem

Search every file with assigned capabilities.

```bash
getcap -r / 2>/dev/null
```

Explanation:

* `-r` → Recursive search
* `/` → Start from the root directory
* `2>/dev/null` → Hide permission errors

Example output:

```text
/usr/bin/python3.11 cap_setuid=ep
/usr/bin/ping cap_net_raw=ep
/usr/bin/gdb cap_sys_ptrace=ep
```

These binaries deserve further investigation.

---

# Method 2 – Check a Running Process

Display the capabilities of a running process.

```bash
getpcaps <PID>
```

Example:

```bash
getpcaps 1234
```

Output:

```text
Capabilities for process 1234:

cap_sys_ptrace
```

---

# Method 3 – Inspect the `/proc` Filesystem

Every running process has information stored under:

```text
/proc/<PID>
```

The file:

```text
/proc/<PID>/status
```

contains capability information.

Example:

```bash
cat /proc/1234/status
```

You may see lines like:

```text
CapEff:
CapPrm:
CapBnd:
```

These values are shown as hexadecimal numbers.

---

# Decoding Capability Values

Linux stores capability sets as hexadecimal bitmasks.

To convert them into readable names, use:

```bash
capsh --decode=<hex_value>
```

Example:

```bash
capsh --decode=0000000000002000
```

Output:

```text
cap_net_raw
```

This makes it much easier to understand which capabilities a process possesses.

---

# Example 1 – Exploiting `cap_setuid`

Imagine the following enumeration result:

```text
/usr/bin/python3.11

↓

cap_setuid
```

This means the Python interpreter can change its User ID.

Since Python can execute arbitrary code, an attacker could write a Python script that:

1. Changes the process's UID to `0` (root).
2. Starts a new shell.

Conceptually, the process looks like this:

```text
Python

↓

Change UID to 0

↓

Launch Bash

↓

Root Shell
```

If successful, the attacker gains a shell running with root privileges.

If the `cap_setuid` capability is removed from the Python binary, the same script will no longer be able to elevate its privileges.

---

# Example 2 – Exploiting `cap_sys_ptrace`

Suppose enumeration reveals:

```text
/usr/bin/gdb

↓

cap_sys_ptrace
```

`gdb` is the GNU Debugger.

Normally, a debugger can inspect and control processes for troubleshooting.

With the `cap_sys_ptrace` capability, it may also be able to attach to processes owned by other users, including root.

Imagine a root-owned process is currently running:

```text
Root Process

↓

Running Normally
```

A debugger with `cap_sys_ptrace` could attach to it and influence its execution.

Conceptually:

```text
Root Process

        ▲

Debugger Attaches

        ▲

Execute Commands

        ▲

Commands Run as Root
```

If the `cap_sys_ptrace` capability is removed, the debugger is no longer allowed to attach to protected processes, and the operation fails.

---

# Why Should You Enumerate Capabilities?

Capabilities are often overlooked during security assessments.

However, a single dangerous capability assigned to the wrong binary can provide a direct path to privilege escalation.

During enumeration, always ask:

* Which binaries have capabilities?
* Are any dangerous capabilities present?
* Can the binary execute arbitrary commands or scripts?
* Can the capability be abused to gain higher privileges?

---

# Best Practices

To reduce security risks:

* Assign only the capabilities an application truly requires.
* Avoid giving powerful capabilities such as `cap_setuid` or `cap_sys_admin` unless absolutely necessary.
* Regularly audit binaries using `getcap -r /`.
* Remove unnecessary capabilities from applications.
* Follow the **Principle of Least Privilege** whenever configuring services.

---

# Key Takeaways

* Linux **Capabilities** divide root privileges into smaller, independent permissions.
* They help implement the **Principle of Least Privilege**, allowing applications to receive only the permissions they actually need.
* Common capabilities include `cap_net_raw`, `cap_setuid`, `cap_sys_admin`, `cap_sys_ptrace`, `cap_sys_module`, `cap_chown`, and `cap_fowner`.
* Use `getcap` to inspect binary capabilities and `setcap` to add or remove them.
* Enumerate capabilities using `getcap -r /`, `getpcaps`, and the `/proc` filesystem.
* Misconfigured capabilities—especially on interpreters, debuggers, or administrative tools—can create serious privilege escalation opportunities.
