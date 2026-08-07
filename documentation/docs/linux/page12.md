# Stack Buffer Overflow and Binary Exploitation Basics

## What You Will Learn

In this guide, you will learn:

* What a stack buffer overflow vulnerability is
* How unsafe memory operations can corrupt program execution
* How security protections prevent exploitation
* The concepts behind NOP sleds, shellcode, and return address overwriting
* How vulnerable SUID binaries can become privilege escalation paths

> **Note:** This topic is focused on understanding memory corruption vulnerabilities and defensive security concepts. Exploitation should only be performed in controlled environments such as security labs or systems you have permission to test.

---

# What is a Buffer Overflow?

A **buffer overflow** occurs when a program writes more data into a memory area than it was designed to store.

A **buffer** is simply a temporary memory space used to store data.

Example:

```c
char buffer[512];
```

This creates a memory area capable of storing:

```text
512 bytes
```

If a program attempts to store:

```text
1000 bytes
```

the extra data has nowhere to go.

Instead, it overwrites nearby memory.

---

# The Vulnerable Program

The demonstration uses a vulnerable C program called:

```text
vulnerable.c
```

The vulnerability exists because the program uses the unsafe function:

```c
strcpy()
```

---

# Understanding `strcpy()`

The purpose of `strcpy()` is to copy one string into another.

Example:

```c
strcpy(destination, source);
```

However, `strcpy()` does **not** check whether the destination buffer is large enough.

Example:

```c
char buffer[512];

strcpy(buffer, user_input);
```

If:

```text
user_input = 1000 bytes
```

the program will still copy everything.

The result:

```text
Input Data

      ↓

512 byte buffer

      ↓

Extra bytes overwrite memory
```

---

# Stack Memory and Program Execution

When a function executes, Linux stores important information on the **stack**.

A simplified stack layout looks like this:

```text
Higher Memory Address

+----------------------+
| Saved Return Address |
+----------------------+
| Saved Frame Pointer  |
+----------------------+
| Local Variables      |
| (buffer)             |
+----------------------+

Lower Memory Address
```

The return address tells the processor:

> Where should the program continue execution after this function finishes?

---

# What Happens During a Buffer Overflow?

When too much input is provided:

```text
Normal:

+----------------+
| Buffer         |
| 512 bytes      |
+----------------+
| Return Address |
+----------------+


Overflow:

+----------------+
| AAAAAAAA       |
| AAAAAAAA       |
+----------------+
| AAAAAAAA       |
+----------------+
```

The extra bytes overwrite important stack data.

Eventually, the attacker can overwrite:

```text
Saved Return Address
```

---

# The Segmentation Fault

A simple test is sending a large amount of data.

Example:

```text
AAAAAAAAAAAAAAAAAAAA...
```

The program may crash.

Why?

Because the return address becomes:

```text
0x41414141
```

(`A` in hexadecimal is `41`)

The CPU tries to continue execution at:

```text
0x41414141
```

but this is not a valid memory location.

The operating system stops the program and produces:

```text
Segmentation fault
```

---

# Modern Security Protections

Modern operating systems include protections that make buffer overflow exploitation much harder.

The demonstration disables these protections to simplify learning.

---

# 1. Executable Stack Protection

## The Problem

Historically, attackers placed malicious instructions directly on the stack.

Modern systems prevent this by marking the stack as:

```text
Non-executable
```

Meaning:

```text
Stack

Can store data

Cannot execute code
```

---

## Disabling the Protection

The program is compiled using:

```bash
-z execstack
```

This allows code stored on the stack to execute.

---

# 2. Stack Canaries

## What Are Stack Canaries?

A stack canary is a random value placed between local variables and the return address.

Example:

```text
+----------------+
| Return Address |
+----------------+
| Canary         |
+----------------+
| Buffer         |
+----------------+
```

Before returning from a function, the program checks:

```text
Is the canary unchanged?
```

If it has been modified:

```text
Stack corruption detected
```

The program terminates.

---

## Disabling Stack Canaries

The demonstration disables them using:

```bash
-fno-stack-protector
```

---

# 3. Address Space Layout Randomization (ASLR)

## What is ASLR?

ASLR randomizes memory addresses every time a program runs.

Without ASLR:

```text
Stack Address:

0xffffd000
```

Every execution uses the same address.

With ASLR:

```text
Run 1:

0xffffd000


Run 2:

0xffa12000


Run 3:

0xffc45000
```

This makes hardcoded memory addresses unreliable.

---

## Disabling ASLR

ASLR can be disabled by setting:

```bash
echo 0 > /proc/sys/kernel/randomize_va_space
```

This makes memory locations predictable.

---

# 4. Using a 32-bit Binary

The demonstration compiles the program as a 32-bit application.

Compilation option:

```bash
-m32
```

Why?

32-bit systems are easier to demonstrate because:

* Addresses are smaller
* Memory layout is simpler
* Addresses are 4 bytes
* Function arguments are stored directly on the stack

Modern systems usually use 64-bit architecture, where exploitation is more complicated.

---

# Building the Exploit

The exploit is created by controlling three important parts:

1. Offset
2. NOP sled
3. Shellcode

---

# Finding the Offset

The offset is the number of bytes required to reach the saved return address.

In this example:

```text
Buffer size:

512 bytes
```

Additional stack data:

```text
12 bytes
```

Total:

```text
512 + 12 = 524 bytes
```

Therefore:

```text
Offset = 524 bytes
```

The payload structure becomes:

```text
[524 bytes padding]

+

[4 byte return address]
```

---

# NOP Sled

## What is a NOP?

NOP means:

```text
No Operation
```

The CPU instruction:

```text
0x90
```

does nothing.

---

## Why Use a NOP Sled?

Memory addresses are difficult to predict.

Instead of jumping to one exact location:

```text
Exact Address
      |
      ▼
Shellcode
```

The exploit creates a large area of NOP instructions:

```text
+----------------+
| NOP            |
| NOP            |
| NOP            |
| NOP            |
+----------------+
| Shellcode      |
+----------------+
```

If execution lands anywhere inside the NOP area:

```text
CPU

↓

NOP

↓

NOP

↓

Shellcode
```

The processor eventually reaches the shellcode.

---

# Shellcode

## What is Shellcode?

Shellcode is a small piece of machine code that performs a specific action.

In this example, the shellcode launches:

```text
/bin/sh
```

The execution flow becomes:

```text
Program

↓

Stack Overflow

↓

Overwrite Return Address

↓

Jump to Shellcode

↓

Execute Shell
```

---

# Return Address Overwrite

The final part of the payload replaces the original return address.

Original:

```text
Return Address

↓

Continue normal program execution
```

Modified:

```text
Return Address

↓

Jump to attacker-controlled memory
```

The attacker redirects program execution.

---

# Creating the Payload

A Python script:

```text
gen_p.py
```

creates the final payload.

The payload contains:

```text
+----------------+
| NOP Sled       |
+----------------+
| Shellcode      |
+----------------+
| Padding        |
+----------------+
| Return Address |
+----------------+
```

The output is saved as:

```text
payload.bin
```

---

# Exploit Execution

When the vulnerable program receives the crafted input:

```text
vulnerable program

        ↓

Reads payload

        ↓

Stack overflow occurs

        ↓

Return address overwritten

        ↓

Execution jumps to shellcode

        ↓

Shell is spawned
```

The attacker gains control of program execution.

---

# Privilege Escalation with SUID Binaries

A normal vulnerable program may only provide access as the current user.

However, the situation becomes much more dangerous when the program is a:

```text
SUID binary
```

---

# What is a SUID Binary?

SUID (Set User ID) allows a program to run with the permissions of its owner.

Example:

```text
File Owner:

root


User Running Program:

attacker
```

The program executes as:

```text
root
```

---

# SUID Exploitation Scenario

Imagine:

```text
/usr/bin/vulnerable

Owner: root

Permission: SUID enabled
```

An attacker exploits the buffer overflow:

```text
Attacker

↓

Control Program Execution

↓

Spawn Shell

↓

Shell Runs as root
```

The attacker gains complete system control.

---

# Why Binary Exploitation Matters

Memory corruption vulnerabilities are still a major security concern.

They can affect:

* Operating systems
* Network services
* Embedded devices
* Applications written in unsafe languages

A single vulnerable privileged program can become a complete system compromise.

---

# Preventing Buffer Overflow Vulnerabilities

Developers can reduce these risks by:

## Use Safe Functions

Avoid:

```c
strcpy()
gets()
sprintf()
```

Prefer safer alternatives:

```c
strncpy()
snprintf()
```

---

## Enable Compiler Protections

Use:

* Stack canaries
* PIE (Position Independent Executables)
* RELRO
* NX protection

---

## Perform Input Validation

Never trust user-controlled input.

Always check:

* Length
* Format
* Data type

---

# Key Takeaways

* A buffer overflow happens when a program writes more data than a memory buffer can store.
* Unsafe functions like `strcpy()` can overwrite nearby stack memory.
* The saved return address controls where execution continues after a function returns.
* By controlling the return address, attackers may redirect execution flow.
* Modern protections such as ASLR, stack canaries, and non-executable stacks make exploitation harder.
* NOP sleds improve reliability by creating a larger landing area before shellcode.
* Shellcode is machine code designed to perform actions such as launching a shell.
* Vulnerable SUID binaries are especially dangerous because successful exploitation can result in root-level access.
