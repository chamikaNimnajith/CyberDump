# PATH Environment Variable and PATH Hijacking 

## 1. What is the PATH Environment Variable?

## Definition of PATH

The **PATH** environment variable is a special variable used by Linux shells to locate executable programs.

When you type a command, Linux needs to find where that program exists on the filesystem.

For example, when you type:

```bash
ls
```

you do not need to type:

```bash
/usr/bin/ls
```

because the shell uses the **PATH variable** to locate the program automatically.

---

# Why Does PATH Exist?

Without PATH, every command would require its complete location.

Example:

Without PATH:

```bash
/usr/bin/python3 script.py
```

With PATH:

```bash
python3 script.py
```

PATH makes command execution faster and more convenient.

---

# Viewing the PATH Variable

You can view the current PATH using:

```bash
echo $PATH
```

Example output:

```text
/usr/local/bin:/usr/bin:/bin
```

This means Linux searches these directories for commands.

---

# PATH Structure

PATH contains multiple directories separated by a colon (`:`).

Example:

```text
/usr/local/bin:/usr/bin:/bin
```

Breaking it down:

```
PATH
 |
 ├── /usr/local/bin
 |
 ├── /usr/bin
 |
 └── /bin
```

Each directory is searched in order.

---

# How Command Resolution Works

Suppose you type:

```bash
ls
```

The shell does not know where `ls` is located.

It checks PATH directories one by one.

Example:

PATH:

```text
/tmp:/usr/local/bin:/usr/bin
```

Search process:

```
1. Check /tmp/ls
        |
        ↓
2. Check /usr/local/bin/ls
        |
        ↓
3. Check /usr/bin/ls
        |
        ↓
Found!
```

The first matching executable is executed.

---

# The which Command

The `which` command shows the exact executable path used by the shell.

Example:

```bash
which ls
```

Output:

```text
/usr/bin/ls
```

This tells us that when we run:

```bash
ls
```

Linux actually executes:

```bash
/usr/bin/ls
```

---

# 2. The Vulnerable Scenario: Reader Binary

To understand PATH hijacking, we first need to understand a vulnerable program.

The example program is called:

```text
Reader
```

---

# What is SUID?

SUID stands for:

**Set User ID**

It is a special Linux permission that allows a program to run with the privileges of its owner.

Normally:

```
User runs program

↓

Program has user's permissions
```

With SUID:

```
User runs program

↓

Program has owner's permissions
```

---

# Example: Root-Owned SUID Program

Suppose:

```bash
ls -l Reader
```

Output:

```text
-rwsr-xr-x 1 root root Reader
```

The important part:

```text
s
```

instead of:

```text
x
```

means SUID is enabled.

The owner is:

```text
root
```

Therefore:

Any user running:

```bash
./Reader
```

executes it with:

```text
root privileges
```

---

# Discovering Program Behavior

Security researchers often analyze binaries to understand what they execute.

One useful command is:

```bash
strings
```

---

# What is strings?

`strings` extracts readable text from binary files.

Example:

```bash
strings Reader
```

Possible output:

```text
cut -Archive %s
```

This reveals that the program may execute:

```bash
cut
```

---

# Program Logic

The source code shows:

```c
system("cut -Archive file");
```

The C function:

```c
system()
```

executes commands through the shell.

Example:

```c
system("ls");
```

is equivalent to:

```bash
ls
```

being executed in the terminal.

---

# The Security Problem

The program executes:

```bash
cut
```

instead of:

```bash
/usr/bin/cut
```

This creates a vulnerability.

Why?

Because Linux must search PATH to find `cut`.

---

# 3. How PATH Hijacking Works

PATH hijacking abuses the way Linux searches for commands.

The attacker tricks a privileged program into executing a malicious program instead of the legitimate one.

---

# Vulnerable Situation

The program:

```
Reader
```

has:

```
SUID root
```

Inside the program:

```bash
cut
```

is executed.

The system searches:

```
PATH
```

to find `cut`.

---

# Attack Steps

## Step 1: Create a Controlled Directory

The attacker chooses a directory they can write to.

Example:

```bash
/tmp
```

---

## Step 2: Modify PATH

The attacker places `/tmp` at the beginning:

Before:

```text
/usr/bin:/bin
```

After:

```text
/tmp:/usr/bin:/bin
```

Command:

```bash
export PATH=/tmp:$PATH
```

---

# Why Put It at the Beginning?

PATH is searched from:

```
left → right
```

Example:

```
PATH=/tmp:/usr/bin
```

The system checks:

First:

```
/tmp/cut
```

Then:

```
/usr/bin/cut
```

Therefore, `/tmp/cut` wins.

---

# Step 3: Create a Fake Program

The attacker creates:

```
/tmp/cut
```

The filename is important.

The vulnerable program expects:

```
cut
```

So the fake file uses the same name.

---

# Step 4: Execute the SUID Program

The attacker runs:

```bash
./Reader
```

The execution flow becomes:

```
Reader (SUID root)

        |
        ↓

system("cut ...")

        |
        ↓

PATH search

        |
        ↓

/tmp/cut found

        |
        ↓

Fake cut executes

        |
        ↓

Runs with root privileges
```

The malicious program inherits the root privileges of:

```
Reader
```

---

# Why Does This Become a Privilege Escalation?

Because:

Before attack:

```
Attacker
 |
 |
Normal user privileges
```

After attack:

```
Attacker
 |
 |
Root privileges
```

The attacker successfully moves from a low-privileged account to administrator access.

---

# Importance of PATH Order

The attack only works if the malicious directory appears first.

Successful:

```
/tmp:/usr/bin:/bin
```

Search:

```
/tmp/cut  ← found first
```

---

Failed:

```
/usr/bin:/bin:/tmp
```

Search:

```
/usr/bin/cut ← found first
```

The real system command executes.

---

# Real-World Impact

PATH hijacking can affect:

* SUID binaries
* Root scripts
* Cron jobs
* Administrative automation scripts

Any privileged program that executes commands without absolute paths may be vulnerable.

---

# 4. Defending Against PATH Hijacking

Developers must prevent privileged programs from trusting user-controlled environments.

---

# Defense 1: Use Absolute Paths

The safest method is specifying the complete path.

Unsafe:

```c
system("cut file.txt");
```

The system searches PATH.

---

Safe:

```c
system("/usr/bin/cut file.txt");
```

Now Linux directly executes:

```
/usr/bin/cut
```

No PATH lookup occurs.

---

# Defense 2: Sanitize the PATH Environment Variable

Privileged programs should not trust user-controlled variables.

Example:

Unsafe PATH:

```
/tmp:/usr/bin:/bin
```

A safer PATH:

```
/usr/bin:/bin
```

Only trusted system directories should be included.

---

# Security Best Practices

When writing privileged programs:

## Avoid:

```bash
command
```

because it depends on PATH.

---

## Prefer:

```bash
/usr/bin/command
```

because the location is controlled.

---

## Validate Environment Variables

Programs should verify:

* PATH
* HOME
* SHELL
* Other user-controlled values

before executing sensitive operations.

---

# Final Summary

| Concept            | Explanation                                        |
| ------------------ | -------------------------------------------------- |
| PATH               | Environment variable that stores command locations |
| Command Resolution | Shell searches PATH directories from left to right |
| which              | Shows the real command path                        |
| SUID               | Executes a program with the owner's privileges     |
| strings            | Extracts readable text from binaries               |
| system()           | Executes shell commands from programs              |
| PATH Hijacking     | Replacing a trusted command with a malicious one   |
| Absolute Path      | Prevents PATH searching                            |
| PATH Sanitization  | Removes untrusted directories                      |

---

# Key Takeaway

PATH hijacking happens because a privileged program trusts the user's environment.

A root-owned SUID program should **never execute commands using only their names**.

Bad:

```bash
cut
```

Good:

```bash
/usr/bin/cut
```

Understanding PATH behavior is important because small configuration mistakes can allow a normal user to gain complete control over a Linux system.
