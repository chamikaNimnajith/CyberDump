
# Linux Commands, File Permissions, and Privilege Management

## 1. Navigating Commands and Documentation (Man Pages)

## What are Man Pages?

Linux systems contain thousands of commands, and remembering every option is impossible.

To solve this problem, Linux provides **manual pages**, commonly called **man pages**.

Man pages are built-in documentation that explains:

* What a command does
* Available options
* Command syntax
* Examples
* Configuration details

Think of man pages as the **official instruction manual for Linux commands**.

---

# Accessing Man Pages

## Using the `man` Command

The basic syntax:

```bash
man <command>
```

Example:

```bash
man ls
```

This opens the manual page for the `ls` command.

---

Example output sections:

```
NAME
    ls - list directory contents

SYNOPSIS
    ls [OPTION]... [FILE]...

DESCRIPTION
    List information about files...
```

---

# Installing Man Pages

Some minimal Linux installations may not include the manual system.

Install it using:

```bash
sudo apt install man-db
```

After installation:

```bash
man command
```

will work.

---

# Online Man Pages

The same documentation can also be accessed online.

Example:

```text
man.org
```

This allows users to search Linux documentation through a browser.

---

# Understanding Man Page Structure

A typical man page contains several sections.

---

## NAME

Provides:

* Command name
* Short description

Example:

```
ls - list directory contents
```

---

## SYNOPSIS

Shows command syntax.

Example:

```
ls [OPTION] [FILE]
```

Meaning:

```text
ls
+
optional arguments
+
file path
```

---

## DESCRIPTION

Explains the command behavior.

Example:

```
ls displays information about files and directories.
```

---

## OPTIONS

Lists available flags.

Example:

```
-a
```

means:

```text
show hidden files
```

So:

```bash
ls -a
```

shows:

```
.bashrc
.config
.hidden_file
```

---

# Why Man Pages Matter in Cybersecurity

During:

* Penetration testing
* CTF challenges
* Linux administration

you frequently encounter unfamiliar commands.

A good security professional first checks:

```bash
man <command>
```

because the documentation may reveal:

* Hidden options
* Dangerous features
* Permission-related behavior
* Security implications

---

Example:

Finding a command that can run as root:

```bash
sudo -l
```

Then checking:

```bash
man command_name
```

may reveal possible privilege escalation paths.

---

# 2. Linux File System Permissions

Linux uses a permission-based security model.

Every file and directory has:

* Owner
* Group
* Permission rules

These permissions decide who can:

* Read
* Modify
* Execute

a resource.

---

# Viewing Permissions

The command:

```bash
ls -l
```

shows detailed file information.

Example:

```
-rwxr-xr-- 1 john developers 2048 script.sh
```

---

A more useful version:

```bash
ls -lha
```

Options:

| Option | Meaning              |
| ------ | -------------------- |
| -l     | Long format          |
| -h     | Human-readable sizes |
| -a     | Show hidden files    |

---

# Understanding the 10-Character Permission String

Example:

```
-rwxr-xr--
```

The permission string contains 10 characters.

---

## First Character: File Type

The first character identifies the resource type.

| Symbol | Meaning       |
| ------ | ------------- |
| -      | Regular file  |
| d      | Directory     |
| l      | Symbolic link |

---

Examples:

Regular file:

```
-rwxr-xr--
```

Directory:

```
drwxr-xr-x
```

Symbolic link:

```
lrwxrwxrwx
```

---

# Remaining Nine Characters

The next nine characters represent permissions.

They are divided into three groups:

```
rwx r-x r--
```

Meaning:

```
Owner | Group | Others
```

---

# User Categories

## Owner (u)

The user who owns the file.

Example:

```
-rwx------
```

The owner has:

```
rwx
```

---

## Group (g)

Users belonging to the file's group.

Example:

```
----r-x---
```

Group members have:

```
r-x
```

---

## Others (o)

Everyone else.

Example:

```
-------r--
```

Other users have:

```
r--
```

---

# Permission Types

## Read (r)

Value:

```
4
```

Allows:

### Files:

Reading content.

Example:

```bash
cat file.txt
```

### Directories:

Listing contents.

Example:

```bash
ls directory/
```

---

## Write (w)

Value:

```
2
```

Allows:

Files:

* Modify content

Directories:

* Create files
* Delete files

---

## Execute (x)

Value:

```
1
```

Files:

* Run programs/scripts

Example:

```bash
./script.sh
```

Directories:

* Enter directory

Example:

```bash
cd folder
```

---

# 3. Modifying Permissions and Ownership

Linux provides commands to modify permissions and ownership.

---

# chmod (Change Mode)

`chmod` changes file permissions.

Syntax:

```bash
chmod permissions file
```

---

# Symbolic Permission Mode

Uses letters:

| Symbol | Meaning    |
| ------ | ---------- |
| u      | User/owner |
| g      | Group      |
| o      | Others     |
| a      | All        |

---

Operators:

| Operator | Meaning           |
| -------- | ----------------- |
| +        | Add permission    |
| -        | Remove permission |
| =        | Set permission    |

---

## Example 1

Add execute permission:

```bash
chmod +x script.sh
```

Result:

Before:

```
rw-r--r--
```

After:

```
rwxr-xr-x
```

---

## Example 2

Remove read and execute permission from others:

```bash
chmod o-rx file.txt
```

---

## Example 3

Give group write access:

```bash
chmod g+w file.txt
```

---

# Numeric Permission Mode

Permissions can also be represented using numbers.

Each permission has a value:

| Permission  | Value |
| ----------- | ----- |
| Read (r)    | 4     |
| Write (w)   | 2     |
| Execute (x) | 1     |

The values are added together.

---

Examples:

## 7 = rwx

```
4 + 2 + 1 = 7
```

Full permission.

---

## 6 = rw-

```
4 + 2 = 6
```

Read and write.

---

## 5 = r-x

```
4 + 1 = 5
```

Read and execute.

---

## 4 = r--

```
4
```

Read only.

---

## 0 = ---

No permissions.

---

# Example: chmod 764

Command:

```bash
chmod 764 script.sh
```

Meaning:

```
7 6 4

Owner  Group  Others
```

Owner:

```
rwx
```

Group:

```
rw-
```

Others:

```
r--
```

Final permission:

```
-rwxrw-r--
```

---

# chown (Change Ownership)

`chown` changes file ownership.

Syntax:

```bash
chown owner file
```

Example:

```bash
sudo chown root script.sh
```

Changes owner to:

```
root
```

---

# Changing Owner and Group

Syntax:

```bash
chown owner:group file
```

Example:

```bash
sudo chown john:developers project.txt
```

Changes:

Owner:

```
john
```

Group:

```
developers
```

---

# Recursive Permission Changes

Both commands support:

```
-R
```

Example:

```bash
sudo chmod -R 755 folder/
```

Applies changes to:

* Folder
* Subfolders
* Files

---

# 4. Special Permission Bits

Linux has additional permissions beyond normal:

* Read
* Write
* Execute

These are called special bits.

---

# Sticky Bit

## Purpose

The sticky bit controls deletion inside shared directories.

Example:

```
/tmp
```

has:

```
drwxrwxrwt
```

The final:

```
t
```

represents sticky bit.

---

Without sticky bit:

Any user with write permission could delete another user's files.

With sticky bit:

Only:

* File owner
* Directory owner
* Root

can delete files.

---

# SUID (Set User ID)

## What is SUID?

SUID allows a program to execute with the permissions of the file owner.

Normally:

```
User runs program

↓

Program gets user's permissions
```

With SUID:

```
User runs program

↓

Program gets owner's permissions
```

---

# Example: passwd Command

The password utility:

```bash
passwd
```

needs to modify:

```
/etc/shadow
```

However:

```
/etc/shadow
```

belongs to:

```
root
```

Normal users cannot edit it.

The solution:

```
passwd
```

has SUID permission.

Example:

```
-rwsr-xr-x
```

The:

```
s
```

means SUID.

When users run:

```bash
passwd
```

it temporarily runs as:

```
root
```

---

# Setting SUID

Command:

```bash
chmod u+s file
```

Example:

```bash
chmod u+s program
```

---

# SGID (Set Group ID)

Similar to SUID but works with groups.

A program runs with the permissions of the file's group.

Set using:

```bash
chmod g+s file
```

---

# 5. The sudo System and Security Risks

## What is sudo?

`sudo` means:

**Super User Do**

It allows normal users to execute commands with higher privileges.

Example:

Normal user:

```bash
apt update
```

may fail.

Using:

```bash
sudo apt update
```

runs as:

```
root
```

---

# Checking sudo Permissions

Command:

```bash
sudo -l
```

Shows what commands the user can execute as root.

Example:

```
User john may run:

(root)
/usr/bin/python3
```

---

# sudoers Configuration

The sudo configuration file:

```
/etc/sudoers
```

controls permissions.

Example:

```
john ALL=(ALL) /usr/bin/vim
```

means:

John can run:

```
vim
```

as root.

---

# Dangerous sudo Configurations

## 1. NOPASSWD

Example:

```
john ALL=(ALL) NOPASSWD: ALL
```

Meaning:

John can run root commands without entering a password.

---

# 2. Wildcards

Example:

```
john ALL=(root) /usr/bin/*
```

This may allow execution of many dangerous programs.

---

# Privilege Escalation Examples

## Python as Root

Dangerous:

```
john ALL=(root) /usr/bin/python3
```

Because Python can execute commands.

Example concept:

```python
import os
os.system("command")
```

---

## Writable Scripts

Example:

Sudo allows:

```
/opt/script.sh
```

but the user can modify it.

Flow:

```
sudo runs script

        ↓

Script executes as root

        ↓

Modified code runs as root
```

---

## Docker Permissions

Docker usually runs with root privileges.

If a user can execute Docker commands as root:

```
sudo docker
```

they may gain administrative control.

---

# Final Summary

| Concept    | Purpose                    |
| ---------- | -------------------------- |
| man        | Command documentation      |
| ls -l      | View permissions           |
| chmod      | Modify permissions         |
| chown      | Change ownership           |
| SUID       | Run program as owner       |
| SGID       | Run program as group       |
| Sticky Bit | Protect shared directories |
| sudo       | Execute commands as root   |
| sudo -l    | View allowed sudo commands |

---

# Key Security Principle

Linux security depends heavily on **correct permissions and privilege management**.

Misconfigured permissions can allow attackers to:

* Read sensitive files
* Modify system files
* Execute commands as root
* Escalate privileges

A cybersecurity professional should always understand:

1. Who owns a resource
2. Who can access it
3. What permissions are assigned
4. Whether those permissions create security risks
