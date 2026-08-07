# Wildcard Expansion (Globbing) and Privilege Escalation

## 1. Understanding Wildcard Expansion (Globbing)

## What is Wildcard Expansion?

**Wildcard expansion (also called globbing)** is a feature of Linux shells such as:

* `bash`
* `sh`
* `zsh`

that allows special characters to represent multiple filenames.

Instead of typing every filename manually, users can use patterns.

Example:

Directory:

```text
/home/user/files/

file1.txt
file2.txt
file3.txt
image.png
```

Command:

```bash
ls *
```

Before executing `ls`, the shell expands `*`:

```bash
ls file1.txt file2.txt file3.txt image.png
```

The command that actually runs contains the full list of matching files.

---

# How Wildcard Expansion Works

The important point:

> The shell expands wildcards **before** the command receives the arguments.

Execution flow:

```
User enters command
        |
        v
Shell expands wildcard
        |
        v
Command receives expanded arguments
        |
        v
Program executes
```

Example:

Input:

```bash
cat *
```

Shell expansion:

```bash
cat file1.txt file2.txt file3.txt
```

Then:

```text
cat receives:
file1.txt
file2.txt
file3.txt
```

The `cat` program does not know that `*` was used.

---

# Types of Wildcards

## 1. Asterisk (*)

The `*` wildcard represents:

> Zero or more characters

Example:

Files:

```
test.txt
test.jpg
test.py
hello.txt
```

Command:

```bash
ls test*
```

Expansion:

```
test.txt
test.jpg
test.py
```

---

## 2. Question Mark (?)

The `?` wildcard represents:

> Exactly one character

Example:

Files:

```
file1.txt
file2.txt
file10.txt
```

Command:

```bash
ls file?.txt
```

Matches:

```
file1.txt
file2.txt
```

Does not match:

```
file10.txt
```

because it contains two characters after `file`.

---

## 3. Square Brackets ([])

Used for character ranges.

Example:

```bash
ls file[123].txt
```

Matches:

```
file1.txt
file2.txt
file3.txt
```

---

## 4. Curly Braces ({})

Used to create multiple choices.

Example:

```bash
echo file{1,2,3}.txt
```

Output:

```
file1.txt
file2.txt
file3.txt
```

---

# 2. The Security Problem with Wildcards

Wildcard expansion becomes dangerous when:

1. A privileged script uses wildcards.
2. A low-privileged user can modify the directory being accessed.

Example:

A root cron job:

```bash
tar -cf backup.tar *
```

runs every minute.

The script owner:

```
root
```

Directory:

```
/backup/
```

But normal users can write:

```
/backup/
```

---

The attacker cannot modify:

```
backup.sh
```

but they can create files inside:

```
/backup/
```

Because the shell expands:

```bash
*
```

the attacker can influence the command arguments.

---

# Core Idea: Argument Injection

Normally:

```
filename.txt
```

is treated as a filename.

However, an attacker creates:

```
--option
```

The program may interpret it as a command option.

Example:

Created file:

```
--help
```

Command:

```bash
program *
```

Shell expands:

```bash
program --help file.txt
```

The program thinks:

```
--help = user supplied option
```

not a filename.

---

## Attack Flow

```
Privileged Script
        |
        v
Uses wildcard (*)
        |
        v
Shell expands filenames
        |
        v
Attacker controls filenames
        |
        v
Program receives malicious options
        |
        v
Privilege escalation
```

---

# Why This Is Similar to Injection

Traditional command injection:

```
User input → command execution
```

Wildcard injection:

```
File names → command arguments
```

The attacker controls the input through filenames.

Therefore it is considered:

* Argument injection
* Command injection variant

---

# 3. Exploitation Examples

---

# A. Exploiting TAR Wildcards

## Scenario

A backup script:

```bash
tar -cf backup.tar *
```

runs as root.

Meaning:

```
root
 |
 +--> tar
       |
       +--> all files in directory
```

---

## TAR Checkpoint Feature

GNU tar has features:

```
--checkpoint
--checkpoint-action
```

They are normally used to execute actions during long archive operations.

Example:

```bash
--checkpoint=1
```

means:

> Perform an action after one file.

---

## Exploitation

The attacker creates files:

```
--checkpoint=1
--checkpoint-action=exec=sh script.sh
```

The directory becomes:

```
file1.txt
file2.txt
--checkpoint=1
--checkpoint-action=exec=sh script.sh
```

---

The script runs:

```bash
tar -cf backup.tar *
```

Shell expands:

```bash
tar -cf backup.tar file1.txt file2.txt \
--checkpoint=1 \
--checkpoint-action=exec=sh script.sh
```

Now tar interprets:

```
--checkpoint
```

and

```
--checkpoint-action
```

as options.

Execution:

```
tar (root)
   |
   v
checkpoint action
   |
   v
attacker script
```

The attacker-controlled command runs with root privileges.

---

# B. Exploiting Find

## Scenario

The `find` command searches files.

Example:

```bash
find /backup -type f -exec cat {} \;
```

Meaning:

```
Find files
     |
     v
Run cat on each file
```

---

## The Problem

The `{}` placeholder is replaced with the filename.

Example:

File:

```
hello.txt
```

Command becomes:

```bash
cat hello.txt
```

---

But attackers can create filenames containing special characters.

Example:

Filename:

```
test; command
```

The semicolon:

```
;
```

has a special meaning in shells.

It separates commands.

Example:

```bash
echo hello; whoami
```

Runs:

```
echo hello
whoami
```

---

## Attack Concept

If a vulnerable script passes filenames to a shell:

```
find
 |
 v
filename injection
 |
 v
extra command execution
```

A malicious filename can add another command.

---

# C. Exploiting rsync

## What is rsync?

`rsync` is a file synchronization tool.

Common uses:

* Backups
* File transfers
* Remote synchronization

Example:

```bash
rsync source destination
```

---

## The Problem

`rsync` supports advanced options.

Examples:

```
-e
--rsync-path
```

These control:

* Remote shell commands
* Helper programs

---

If a privileged backup script uses:

```bash
rsync *
```

and attackers control filenames:

```
--option
```

they can inject rsync options.

---

Execution flow:

```
Root rsync backup
        |
        v
Wildcard expansion
        |
        v
Malicious filename becomes option
        |
        v
rsync executes attacker-controlled command
```

---

# 4. Important Security Lessons

## File Name vs File Content

A common misunderstanding:

> The malicious code is inside the file.

Wrong.

In wildcard attacks:

```
File content ❌
File name ✅
```

The attacker does not need to modify the file content.

The filename itself becomes the payload.

Example:

Normal:

```
backup.txt
```

Malicious:

```
--checkpoint-action=exec=script.sh
```

---

# Prevention and Mitigation

## 1. Avoid Wildcards in Privileged Scripts

Dangerous:

```bash
tar backup.tar *
```

Safer:

```bash
tar backup.tar file1 file2 file3
```

---

## 2. Use Secure Directories

Avoid running privileged scripts in:

```
/tmp
```

or directories writable by normal users.

Example:

Danger:

```
root cron job
        |
        v
/tmp/*
```

Any user can create files.

---

## 3. Use Absolute Paths

Avoid:

```bash
backup *
```

Prefer:

```bash
/usr/bin/tar -cf backup.tar /secure/files/*
```

---

## 4. Prevent Option Injection

Many tools support:

```
--
```

which means:

> Stop processing options.

Example:

```bash
tar -cf backup.tar -- *
```

Now filenames beginning with:

```
-
```

are treated as filenames instead of options.

---

# Summary Table

| Concept              | Explanation                                     |
| -------------------- | ----------------------------------------------- |
| Wildcard Expansion   | Shell replaces patterns with matching filenames |
| Globbing             | Another name for wildcard expansion             |
| `*`                  | Matches zero or more characters                 |
| `?`                  | Matches one character                           |
| Argument Injection   | Filename becomes a command option               |
| Root Cron + Wildcard | Common privilege escalation scenario            |
| TAR Attack           | Abuse checkpoint options                        |
| Find Attack          | Abuse filename execution with `-exec`           |
| rsync Attack         | Abuse injected rsync options                    |
| Main Payload         | Malicious filename, not file content            |

---

