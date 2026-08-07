# Windows Binary Compilation and Cross-Compilation Fundamentals

## Introduction

When developing software or security tools for Windows, understanding how programs are compiled and executed is very important.

A program written in a high-level language such as **C** is not directly understood by the computer. It must first be converted into a binary format containing machine instructions that the operating system can load and execute.

Different operating systems use different binary formats, which means a program compiled for one operating system may not work on another.

This note explains:

* How compilation works
* Difference between Linux ELF and Windows PE binaries
* Cross-compilation using mingw-w64
* Static and dynamic linking
* Creating Windows-compatible binaries from Linux
* Basic concepts behind custom Windows programs

---

# 1. Compilation Fundamentals (ELF vs PE)

## What is Compilation?

Compilation is the process of converting human-readable source code into machine-executable instructions.

Example:

A C program:

```c
#include <stdio.h>

int main()
{
    printf("Hello World");
    return 0;
}
```

is understandable by humans but not directly by the CPU.

The compiler converts it:

```text
C Source Code
      |
      ↓
Compiler
      |
      ↓
Machine Instructions
      |
      ↓
Executable Binary
```

The final executable contains instructions that the CPU can execute.

---

# The Binary Wrapper

A compiled program contains two important parts:

1. Machine instructions
2. A binary format structure

The binary format tells the operating system:

* How to load the program
* Where memory should be allocated
* What libraries are required
* Where execution starts

Think of it like a package:

```
Executable File

+----------------+
| Binary Header  |
+----------------+
| Machine Code   |
+----------------+
| Data Sections  |
+----------------+
```

The operating system reads the header first before running the program.

---

# Compilation Environment

A binary depends on two main factors:

## 1. CPU Architecture

Examples:

* x86
* x64
* ARM
* ARM64

A binary compiled for ARM cannot normally run on an x64 processor.

Example:

```
ARM Binary
     |
     X
     |
x64 CPU
```

---

## 2. Operating System

The operating system also determines the executable format.

Examples:

Linux:

```
ELF
(Executable and Linkable Format)
```

Windows:

```
PE
(Portable Executable)
```

---

# Linux ELF Format

Linux uses:

```
ELF Executables
```

Example:

```bash
gcc hello.c -o hello
```

Checking:

```bash
file hello
```

Output:

```
ELF 64-bit LSB executable
```

Linux understands this format.

---

# Windows PE Format

Windows uses:

```
Portable Executable (PE)
```

Examples:

```
.exe
.dll
.sys
```

Checking:

```
file hello.exe
```

Output:

```
PE32+ executable for MS Windows
```

---

# Why Linux Binaries Do Not Run on Windows

Example:

A C program compiled on Linux:

```bash
gcc hello.c -o hello
```

creates:

```
hello
|
|
↓
ELF executable
```

Copying it to Windows:

```
Windows
   |
   ↓
Cannot understand ELF format
```

The program fails because Windows expects:

```
PE executable
```

not:

```
ELF executable
```

---

# 2. Cross-Compilation Using mingw-w64

## What is Cross-Compilation?

Cross-compilation means:

> Compiling software on one operating system but creating a binary for another operating system.

Example:

Develop on Linux:

```
Linux Machine
      |
      |
      ↓
Cross Compiler
      |
      ↓
Windows .exe File
```

---

# mingw-w64

`mingw-w64` is a cross-compilation toolchain that allows Linux systems to create Windows PE binaries.

It provides Windows-compatible compilers.

---

# Installing mingw-w64

## Debian / Ubuntu

```bash
sudo apt install mingw-w64
```

---

## Arch Linux

```bash
sudo pacman -S mingw-w64-gcc
```

---

# Available Compilers

After installation:

## Windows 64-bit C Compiler

```bash
x86_64-w64-mingw32-gcc
```

Used for:

```
C → Windows x64 executable
```

---

## Windows 64-bit C++ Compiler

```bash
x86_64-w64-mingw32-g++
```

Used for:

```
C++ → Windows x64 executable
```

---

# Finding Target Architecture

On Windows:

```cmd
systeminfo
```

Look for:

```
System Type
```

Example:

```
x64-based PC
```

means:

```
64-bit Windows
```

Therefore, compile:

```
x86_64
```

---

# 3. Static vs Dynamic Linking

When a program uses external libraries, the compiler must decide how those libraries are included.

There are two approaches:

1. Dynamic Linking
2. Static Linking

---

# Dynamic Linking

Dynamic linking means:

> The executable uses libraries already installed on the operating system.

Example:

Windows:

```
program.exe
      |
      ↓
kernel32.dll
user32.dll
ws2_32.dll
```

Linux:

```
program
      |
      ↓
libc.so
libpthread.so
```

The executable only stores references.

Advantages:

* Smaller file size
* Shared libraries save disk space

Disadvantages:

* Missing DLLs can break execution

Example:

```
Program starts

↓
Looking for missing.dll

↓
Error:
DLL not found
```

---

# Checking Dynamic Dependencies

Linux:

```bash
ldd program
```

Example:

```
libc.so.6
libpthread.so
```

---

# Static Linking

Static linking copies required libraries directly into the executable.

Example:

```
Executable

+----------------+
| Program Code   |
+----------------+
| Library Code   |
+----------------+
```

Advantages:

* No external dependencies
* More portable

Disadvantages:

* Larger file size

---

# Static Linking During Windows Cross Compilation

Example:

```bash
x86_64-w64-mingw32-gcc hello.c -o hello.exe -static
```

The `-static` option includes libraries inside the executable.

This reduces problems caused by missing DLL files.

---

# 4. Practical Example: Hello World Program

## Linux Compilation

Source:

```
hello.c
```

Compile:

```bash
gcc hello.c -o hello
```

Check:

```bash
file hello
```

Output:

```
ELF 64-bit executable
```

This runs on Linux.

---

# Windows Cross Compilation

Compile:

```bash
x86_64-w64-mingw32-gcc hello.c -o hello.exe -static
```

Check:

```bash
file hello.exe
```

Output:

```
PE32+ executable for MS Windows
```

Now Windows understands the executable.

---

# Transferring to Windows

The file can then be copied to a Windows machine.

Example:

```
Linux
 |
 |
 ↓
hello.exe
 |
 |
 ↓
Windows VM
```

Running:

```cmd
hello.exe
```

Output:

```
Hello World
```

---

# 5. Creating a Custom Windows Program

A more advanced example is creating a Windows program that starts a PowerShell connection.

The program:

* Receives IP address
* Receives port number
* Creates a PowerShell command
* Executes it

---

# Program Structure

A C program usually starts with:

```c
int main(int argc, char *argv[])
```

---

# Argument Handling

`argc` stores the number of arguments.

Example:

```
program.exe 192.168.1.5 7777
```

Arguments:

```
argv[0] = program name

argv[1] = IP address

argv[2] = Port
```

The program checks:

```
Are enough arguments provided?
```

If not:

```
Show usage information
Exit
```

---

# Pointer Usage

The program accesses values stored inside:

```
argv[]
```

Example:

```
argv[1]
 |
 ↓
192.168.1.5
```

The program reads these values and uses them later.

---

# Input Validation

Good software should validate user input.

Two important checks:

## IP Address Validation

Windows provides APIs for network validation.

Example:

```
WSAStringToAddress()
```

This checks whether the supplied address is valid.

---

## Port Validation

Ports must be between:

```
1 - 65535
```

The program converts:

```
"7777"
```

into:

```
7777
```

using:

```
atoi()
```

Then verifies the range.

---

# Generating Commands Safely

The program creates a command template.

Example concept:

```
PowerShell Command Template

        +
        
User supplied IP and Port

        ↓

Final Command
```

To avoid security issues:

* Calculate required memory size
* Allocate memory safely
* Use secure formatting functions

Example:

```
snprintf()
```

is safer than:

```
sprintf()
```

because it limits the amount of data written.

---

# Executing the Generated Command

The program writes the command to a file and executes it.

The execution flow:

```
C Program

     ↓

Creates Script

     ↓

Writes File

     ↓

system()

     ↓

Windows Executes Command
```

---

# Compiling the Windows Program

Because the program uses Windows networking APIs, extra libraries are required.

Example:

```bash
x86_64-w64-mingw32-gcc shell.c -o shell.exe -static -lws2_32
```

Explanation:

| Option                 | Purpose                      |
| ---------------------- | ---------------------------- |
| x86_64-w64-mingw32-gcc | Windows compiler             |
| -o                     | Output filename              |
| -static                | Include libraries            |
| -lws2_32               | Link Windows Winsock library |

---

# Winsock Library

Windows networking functions are provided by:

```
Winsock
```

Library:

```
ws2_32.dll
```

It provides:

* Socket communication
* Network connections
* IP handling

---

# Running the Program

The general workflow:

```
Attacker Machine

Compile Program

        ↓

Transfer EXE

        ↓

Windows Machine

        ↓

Execute Program

        ↓

Program performs its task
```

---

# Limitations of Simple Programs

A basic program may work in a lab environment but has several weaknesses.

---

# 1. No Persistence

If the connection stops:

```
Connection Closed
        |
        ↓
Access Lost
```

A stronger design would include recovery mechanisms.

Example:

```
Try connection

If failed:

Wait

Try again
```

---

# 2. High Visibility

Simple programs create noticeable activity:

* New executable files
* Network connections
* Command execution

Security software can detect these behaviors.

---

# 3. Disk Activity

A program that writes scripts to disk creates artifacts:

Example:

```
script.ps1
temporary files
logs
```

Security tools monitor these changes.

More advanced software often attempts to reduce unnecessary disk activity by using memory-based execution techniques.

---

# Summary

Understanding Windows executable formats and compilation is essential for software development and security research.

Key concepts:

| Concept                | Explanation                            |
| ---------------------- | -------------------------------------- |
| Compilation            | Converts source code into machine code |
| ELF                    | Linux executable format                |
| PE                     | Windows executable format              |
| Cross Compilation      | Building software for another OS       |
| mingw-w64              | Linux-to-Windows compiler toolchain    |
| Static Linking         | Includes libraries inside executable   |
| Dynamic Linking        | Uses external libraries                |
| Winsock                | Windows networking API                 |
| x86_64-w64-mingw32-gcc | Windows 64-bit compiler                |

The main idea:

```
Source Code
     |
     ↓
Compiler
     |
     ↓
Operating System Specific Binary
     |
     ↓
Executable Program
```

A Linux compiler creates Linux-compatible ELF files, while tools like **mingw-w64** allow developers and security professionals to create Windows-compatible PE executables from a Linux environment. Understanding this process is important for Windows administration, software development, malware analysis, and penetration testing.
