# SUDO Fundamentals and Privilege Escalation

## 1. Understanding SUDO

## What is SUDO?

**SUDO** stands for:

> **SuperUser Do**

`sudo` is a Linux utility that allows a permitted user to execute commands with the privileges of another user.

Most commonly, it is used to execute commands as:

```text
root user (UID 0)
```

However, `sudo` is not limited to root. A user can execute commands as **any permitted user**.

Example:

```bash
sudo -u username command
```

This means:

> Run the command as another user.

---

## Normal Command Execution

Without sudo:

```bash
apt update
```

The command runs with the current user's permissions:

```
User → Command
       |
       v
  Normal privileges
```

---

## Using SUDO

Example:

```bash
sudo apt update
```

Execution:

```
Normal User
      |
      v
     sudo
      |
      v
    root privileges
```

The command runs with elevated permissions.

---

# 2. Checking SUDO Permissions

## The sudo -l Command

The most important command for checking sudo privileges is:

```bash
sudo -l
```

It displays:

* Which commands the user can run with sudo
* Which users they can impersonate
* Whether a password is required

Example output:

```
User alice may run the following commands:

(root) /usr/bin/vim
(root) /usr/bin/python3
```

Meaning:

Alice can run:

```bash
sudo vim
sudo python3
```

as root.

---

# Why sudo -l Requires a Password

When executing:

```bash
sudo -l
```

Linux asks:

```
[sudo] password for alice:
```

This verifies that the user actually owns the account.

---

## Security Impact During Penetration Testing

Imagine an attacker obtains a reverse shell:

```
Attacker
   |
   v
Reverse shell as user alice
```

The attacker tries:

```bash
sudo -l
```

But:

```
[sudo] password for alice:
```

appears.

Without the user's password:

```
sudo abuse ❌
```

The attacker must first:

* Obtain the password
* Find credentials
* Perform credential attacks

before abusing sudo privileges.

---

# 3. The SUDOERS Configuration File

The main sudo configuration file is:

```bash
/etc/sudoers
```

It controls:

* Who can use sudo
* Which commands are allowed
* Which users they can impersonate

---

# Editing sudoers Safely

Never edit:

```bash
/etc/sudoers
```

directly.

Instead use:

```bash
visudo
```

---

## Why use visudo?

`visudo` checks the file for syntax errors before saving.

Example:

Wrong configuration:

```
alice ALL=(ALL) /bin/bash
```

A typo can break sudo completely.

Without `visudo`:

```
sudo stops working
```

With `visudo`:

```
Syntax error detected
Changes rejected
```

---

# 4. Sudoers File Structure

A sudo rule follows this format:

```
user HOST=(run_as_user:run_as_group) command
```

Example:

```
alice ALL=(root) /usr/bin/vim
```

Meaning:

| Part         | Meaning                   |
| ------------ | ------------------------- |
| alice        | User receiving permission |
| ALL          | Any host                  |
| root         | User to impersonate       |
| /usr/bin/vim | Allowed command           |

---

# Example: Full Root Access

Default root rule:

```
root ALL=(ALL:ALL) ALL
```

Explanation:

| Section   | Meaning                       |
| --------- | ----------------------------- |
| root      | Applies to root user          |
| ALL       | Any machine                   |
| (ALL:ALL) | Can become any user and group |
| ALL       | Can run any command           |

Meaning:

> Root can execute anything as anyone.

---

# Group Permissions

Groups use `%`.

Example:

```
%sudo ALL=(ALL:ALL) ALL
```

Meaning:

> Anyone inside the sudo group can run any command as any user.

Check groups:

```bash
groups username
```

Example:

```
alice sudo users
```

Alice can use sudo.

---

# NOPASSWD Option

Normally:

```bash
sudo command
```

requires a password.

Example:

```
alice ALL=(root) /usr/bin/vim
```

When Alice runs:

```bash
sudo vim
```

Linux asks:

```
Password:
```

---

With:

```
alice ALL=(root) NOPASSWD:/usr/bin/vim
```

The password is not required.

Now:

```bash
sudo vim
```

runs immediately.

---

# 5. Important SUDO Security Settings

## env_reset

Example:

```
Defaults env_reset
```

Purpose:

> Remove unsafe environment variables before running sudo commands.

This prevents attacks where attackers manipulate environment variables.

---

## secure_path

Example:

```
Defaults secure_path="/usr/local/bin:/usr/bin:/bin"
```

Purpose:

Controls the PATH used by sudo commands.

It prevents:

```
PATH hijacking attacks
```

Example attack:

A user creates:

```
/tmp/ls
```

If sudo searches `/tmp` first:

```
sudo ls
     |
     v
/tmp/ls (malicious)
```

The attacker executes code as root.

`secure_path` prevents this.

---

# 6. SUDO Privilege Escalation

## Why SUDO Can Be Dangerous

Sudo itself is not vulnerable.

The problem is:

```
Overly powerful sudo permissions
        +
Dangerous programs
        =
Privilege escalation
```

Example:

A user can run:

```
sudo vim
```

as root.

Since Vim can execute shell commands:

```
vim
 |
 v
root shell
```

---

# GTFOBins

Security researchers use:

**GTFOBins**

to find ways that legitimate Linux binaries can be abused when they have:

* sudo permissions
* SUID permissions

Website:

```
gtfobins.github.io
```

It contains examples for binaries like:

* vim
* python
* tar
* awk
* find
* less
* base64

---

# 7. SUDO Exploitation Examples

---

# Example 1: Switching Users (Privilege Pivot)

## Scenario

A user:

```
www-data
```

has permission:

```
sudo -u script-manager ALL
```

Meaning:

```
www-data
        |
        v
script-manager
```

The user can become:

```
script-manager
```

without a password.

---

Command:

```bash
sudo -u script-manager python3 -c 'import pty; pty.spawn("/bin/bash")'
```

Explanation:

* `sudo -u script-manager` → run as another user
* `python3` → starts Python
* `pty.spawn()` → creates an interactive shell

Result:

Before:

```
www-data
```

After:

```
script-manager
```

This is called:

> Privilege pivoting

---

# Example 2: Exploiting PIP Installation

## Scenario

The user can run:

```
sudo pip install *
```

as root.

The problem:

`pip install` executes Python package installation scripts.

---

## Attack Idea

Create a malicious Python package.

Structure:

```
malicious_package/

 ├── setup.py
 └── package files
```

The file:

```
setup.py
```

contains code that executes during installation.

Example flow:

```
sudo pip install package
             |
             v
        setup.py runs
             |
             v
       attacker code
             |
             v
            root
```

---

Command:

```bash
sudo /usr/bin/pip install . \
--upgrade \
--force-reinstall \
--break-system-packages
```

The installation runs as root.

The malicious setup script can:

* Read protected files
* Copy SSH keys
* Execute commands

Example:

```
/root/.ssh/id_rsa
        |
        v
/tmp/id_rsa
```

The attacker can then use the stolen key.

---

# Example 3: Exploiting TAR Checkpoints

## Scenario

User can execute:

```
sudo tar
```

with unsafe options.

GNU tar has checkpoint functionality:

Example:

```
--checkpoint-action
```

It allows commands to run during archive operations.

Attack flow:

```
sudo tar
   |
   v
checkpoint trigger
   |
   v
execute command
   |
   v
root shell
```

Because tar runs as root, the command also runs as root.

---

# Example 4: Reading Files Using Base64

## Scenario

The user can execute:

```
sudo base64
```

---

Normally:

```bash
cat /etc/shadow
```

fails:

```
Permission denied
```

because:

```
/etc/shadow → root only
```

---

But:

```bash
sudo base64 /etc/shadow
```

runs as root.

The file is converted:

```
Sensitive file
       |
       v
Base64 encoded output
```

Decode:

```bash
base64 -d
```

Result:

```
Original sensitive data
```

---

Possible targets:

```
/etc/shadow
/root/.ssh/id_rsa
/root/.bash_history
```

---

# Summary Table

| Concept         | Explanation                                        |
| --------------- | -------------------------------------------------- |
| SUDO            | Execute commands with another user's privileges    |
| sudo -l         | Shows allowed sudo commands                        |
| /etc/sudoers    | Configuration file controlling sudo permissions    |
| visudo          | Safe editor with syntax checking                   |
| NOPASSWD        | Allows sudo without password                       |
| env_reset       | Removes unsafe environment variables               |
| secure_path     | Prevents PATH manipulation                         |
| GTFOBins        | Database of binary privilege escalation techniques |
| Privilege pivot | Moving from one user account to another            |
| Dangerous sudo  | Allowing powerful binaries as root                 |

---


