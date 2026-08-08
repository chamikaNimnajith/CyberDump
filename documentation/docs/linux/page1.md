

# Linux Command Line Fundamentals 

##  Command Line Interface (CLI) vs Graphical User Interface (GUI)

## What is a GUI?

A **Graphical User Interface (GUI)** allows users to interact with computers through visual elements such as:

* Windows
* Icons
* Buttons
* Menus
* Mouse interactions

Examples:

* Windows Desktop
* Ubuntu Desktop Environment
* File managers

Example action:

```
Open a folder
        ↓
Double click icon
        ↓
Navigate files visually
```

GUIs are beginner-friendly because users do not need to remember commands.

---

# Limitations of GUI

Although GUIs are convenient, they have limitations.

## 1. Manual Interaction

GUIs require human actions.

Example:

Deleting 100 files:

GUI method:

```
Select file
Click delete
Confirm
Repeat 100 times
```

This is slow and inefficient.

---

## 2. Difficult to Automate

A GUI requires:

* Mouse movements
* Button clicks
* Visual decisions

Automation becomes complicated.

Example:

A security analyst wants to:

* Scan 1000 machines
* Collect logs
* Analyze results

Using GUI tools manually is inefficient.

---

# What is a CLI?

A **Command Line Interface (CLI)** is a text-based way to communicate with an operating system.

Instead of clicking:

```
Open browser
```

we type:

```bash
firefox
```

Instead of opening a folder:

```bash
cd Documents
```

---

# Advantages of CLI

## 1. Automation

Commands can be placed inside scripts.

Example:

```bash
#!/bin/bash

echo "Starting scan"
nmap target.com
echo "Finished"
```

The computer executes tasks automatically.

---

## 2. Faster Repetitive Operations

Example:

Finding all log files:

GUI:

```
Open folders
Search manually
Check results
```

CLI:

```bash
find / -name "*.log"
```

---

## 3. Better Control

CLI exposes more advanced operating system functions.

Cybersecurity professionals use CLI for:

* Network analysis
* System administration
* Penetration testing
* Automation

---

# Relationship Between GUI and CLI

A common misconception is that GUI and CLI are separate systems.

Actually:

```
GUI
 |
 |
Uses
 |
 ↓
CLI / System Commands
 |
 ↓
Operating System
```

Many GUI applications execute commands internally.

The CLI is closer to the operating system.

---

# 3. Understanding the SSH Command Structure

## What is SSH?

**SSH (Secure Shell)** is a protocol used to remotely access another computer securely.

Common uses:

* Managing servers
* Remote administration
* Penetration testing
* Cloud server access

---

# SSH Client-Server Model

SSH works using two components:

```
SSH Client
(Your machine)

        |
        |
Encrypted TCP Connection

        |
        |

SSH Server
(Remote machine)
```

---

# Basic SSH Command

Example:

```bash
ssh username@IP_address
```

Example:

```bash
ssh john@192.168.1.10
```

Meaning:

```
username = john

IP address = 192.168.1.10
```

---

# SSH Port

SSH normally uses:

```
TCP Port 22
```

Example:

```
Client
 |
 | TCP 22
 |
Server
```

---

# Connecting to a Different Port

If SSH runs on another port:

Example:

```
SSH Port = 77
```

Command:

```bash
ssh -p 77 username@IP
```

The `-p` option changes the default port.

---

# SSH Options / Flags

Commands often have options that modify behavior.

Example:

```bash
ssh -o UserKnownHostsFile=/dev/null user@host
```

---

## UserKnownHostsFile

Normally SSH stores server fingerprints in:

```
~/.ssh/known_hosts
```

This prevents man-in-the-middle attacks.

Using:

```bash
-o UserKnownHostsFile=/dev/null
```

prevents permanent storage of the fingerprint.

---

# 4. Terminal, Shell, and TTY

These three terms are often confused.

They perform different roles.

---

# Terminal

A **terminal** is the graphical application where users type commands.

Examples:

* xterm
* GNOME Terminal
* VTerm

It provides:

* Window display
* Keyboard input
* Visual output

Example:

```
User
 |
 ↓
Terminal Application
```

---

# Shell

A **shell** is the command interpreter.

It reads commands and executes them.

Examples:

* bash
* sh
* zsh
* fish

Flow:

```
User types:

ls

↓

Shell interprets command

↓

Operating System executes it
```

---

# TTY

TTY stands for:

**Teletypewriter**

A TTY is the communication channel between:

```
Terminal
   |
   |
   ↓
TTY
   |
   |
   ↓
Shell
```

It manages:

* Input
* Output
* Communication

---

# Example

When you open a terminal:

```
Terminal Program
        |
        ↓
       TTY
        |
        ↓
      Bash Shell
        |
        ↓
    Linux Kernel
```

---

# 5. Basic System Information Commands

Before performing system administration or security testing, you need information about the machine.

---

# whoami

Displays the current username.

Command:

```bash
whoami
```

Example output:

```
chamika
```

---

# id

Shows detailed user information.

Command:

```bash
id
```

Example:

```
uid=1000(chamika)
gid=1000(chamika)
groups=1000(chamika)
```

Information:

* UID → User ID
* GID → Group ID
* Groups → Permissions

---

# hostname

Shows the computer name.

Command:

```bash
hostname
```

Example:

```
ubuntu-server
```

---

# Shell Prompt

The command prompt provides context.

Example:

```bash
chamika@ubuntu:~$
```

Meaning:

```
chamika = username

ubuntu = hostname

~ = current directory
```

---

Some shells have customized prompts.

Example:

Bash:

```
user@machine:~$
```

Simple sh shell:

```
$
```

---

# pwd

Shows the current directory.

Command:

```bash
pwd
```

Example:

```
/home/chamika/Documents
```

---

# Home Directory

Linux uses:

```
~
```

as a shortcut for the current user's home.

Example:

```bash
cd ~
```

equals:

```bash
cd /home/user
```

---

# Environment Variables

Environment variables store system information.

View all:

```bash
env
```

Example:

```
HOME=/home/user
SHELL=/bin/bash
PATH=/usr/bin
```

---

## Accessing Variables

Use:

```bash
echo
```

Example:

```bash
echo $SHELL
```

Output:

```
/bin/bash
```

---

# PATH Variable

PATH tells the shell where to search for programs.

Example:

```
PATH=/usr/bin:/bin
```

When you type:

```bash
ls
```

Linux searches:

```
/usr/bin/ls
```

---

# which

Shows the location of a command.

Example:

```bash
which python
```

Output:

```
/usr/bin/python
```

---

# 6. File System Paths and Directory Commands

Linux uses a hierarchical file system.

Example:

```
/
|
├── home
|
├── etc
|
├── var
|
└── bin
```

---

# Absolute Path

Starts from root:

```
/
```

Example:

```
/home/user/Documents/file.txt
```

Complete location.

---

# Relative Path

Starts from the current directory.

Example:

Current location:

```
/home/user
```

Command:

```bash
cd Documents
```

Moves to:

```
/home/user/Documents
```

---

# Parent Directory

Move backward:

```bash
cd ..
```

Example:

Before:

```
/home/user/Documents
```

After:

```
/home/user
```

---

# cd

Change directory.

Example:

```bash
cd /etc
```

---

# ls

List files.

Basic:

```bash
ls
```

---

## ls -a

Shows hidden files.

Linux hidden files start with:

```
.
```

Example:

```
.bashrc
.config
```

---

## ls -al

Detailed listing:

Shows:

* Permissions
* Owner
* Size
* Date

---

## ls -alh

Human-readable sizes:

Example:

```
5K
20M
1G
```

---

# cat

Displays file contents.

Example:

```bash
cat file.txt
```

Multiple files:

```bash
cat file1 file2
```

---

# mv

Move or rename files.

Rename:

```bash
mv old.txt new.txt
```

Move:

```bash
mv file.txt /tmp/
```

---

# cp

Copy files.

Example:

```bash
cp file.txt backup.txt
```

Original remains.

---

# rm

Remove files.

Example:

```bash
rm file.txt
```

Important:

```
rm is dangerous
```

because deleted files are not moved to a recycle bin.

---

# 7. Operating System Resource Management

System enumeration is important for:

* Administration
* Troubleshooting
* Privilege escalation

---

# Disk Information

## fdisk

Displays partitions.

Command:

```bash
sudo fdisk -l
```

Shows:

* Drives
* Partitions
* Sizes

---

# df

Shows disk usage.

Example:

```bash
df -h
```

Output:

```
Filesystem
Used
Available
```

---

# du

Shows folder size.

Example:

```bash
du -h Documents
```

---

# Process Management

## ps

Shows running processes.

Basic:

```bash
ps
```

---

## ps aux

Shows all processes.

Important for security:

```
Which services are running?
Which user owns them?
```

Example:

```
root     sshd
user     firefox
```

---

# Network Enumeration

## ip address

Shows network interfaces.

Shortcut:

```bash
ip a
```

Displays:

* IP addresses
* Network adapters

---

# netstat

Shows network connections.

Example:

```bash
netstat -ltnp
```

Displays:

* Listening ports
* TCP connections
* Running services

---

# Command Help

Most Linux commands support:

```bash
--help
```

Example:

```bash
netstat --help
```

Shows usage information.

---

# 8. User and Group Management

Linux uses users and groups to control permissions.

---

# sudo

Runs commands as administrator.

Example:

```bash
sudo apt update
```

---

# Creating Users

Command:

```bash
sudo useradd -m username
```

Creates:

```
/home/username
```

---

# Important User Files

## /etc/passwd

Contains user information:

```
username
UID
home directory
shell
```

Example:

```
john:x:1001:/home/john:/bin/bash
```

---

## /etc/shadow

Stores password hashes.

Example:

```
$6$encrypted_password
```

Only root can read it.

---

# passwd

Change password:

```bash
passwd username
```

---

# su

Switch user.

Example:

```bash
su john
```

---

# userdel

Delete user:

```bash
sudo userdel -r john
```

The `-r` removes the home directory.

---

# Groups

View groups:

```bash
groups
```

Create group:

```bash
groupadd developers
```

Add user:

```bash
usermod -aG developers john
```

---

# 9. Linux Package Management (APT)

## What is APT?

APT is Ubuntu's package management system.

It manages:

* Installing software
* Updating software
* Removing software

---

# Check Linux Version

Command:

```bash
lsb_release -a
```

Example:

```
Ubuntu 24.04
```

---

# Install Software

Example:

```bash
sudo apt install nmap
```

APT automatically installs dependencies.

---

# Search Packages

```bash
apt search keyword
```

Example:

```bash
apt search apache
```

---

# Remove Software

Remove program:

```bash
sudo apt purge package
```

Also removes configuration files.

---

# apt update vs apt upgrade

## apt update

Downloads latest package information.

It does:

```
Repository
      |
      ↓
Update package database
```

---

## apt upgrade

Actually installs newer versions.

Flow:

```
apt update

↓

apt upgrade

↓

New software versions installed
```

---

# 10. Practice Resources

Learning Linux CLI requires practice.

---

# OverTheWire Bandit

A command-line based security challenge.

Website:

```
overthewire.org
```

It teaches:

* Linux commands
* File permissions
* SSH
* Searching files
* Privilege concepts

---

# Hack The Box

Provides:

* Realistic machines
* Penetration testing practice
* Security challenges

---

# TryHackMe

Beginner-friendly cybersecurity learning platform.

Topics:

* Linux
* Networking
* Web security
* Privilege escalation

---

# OSCP Labs

Advanced penetration testing practice environment.

Focuses on:

* Enumeration
* Exploitation
* Privilege escalation

---

# Final Summary

| Concept       | Purpose                     |
| ------------- | --------------------------- |
| CLI           | Text-based system control   |
| GUI           | Visual interaction          |
| SSH           | Secure remote access        |
| Terminal      | User interface for commands |
| Shell         | Command interpreter         |
| TTY           | Communication channel       |
| whoami        | Current user                |
| id            | User details                |
| pwd           | Current directory           |
| ls            | List files                  |
| ps            | Processes                   |
| ip a          | Network information         |
| sudo          | Administrative execution    |
| passwd/shadow | User authentication         |
| apt           | Software management         |

---

## Key Takeaway

For cybersecurity professionals, the command line is not just a faster way to operate a computer — it is a fundamental skill.

Understanding the CLI allows you to:

* Automate tasks
* Analyze systems
* Troubleshoot problems
* Perform security assessments
* Understand how operating systems actually work

A strong security professional does not only know commands; they understand **what happens internally when those commands are executed**.
