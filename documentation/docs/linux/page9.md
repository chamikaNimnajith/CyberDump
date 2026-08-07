# Linux Cron Jobs and Privilege Escalation

## What You Will Learn

In this guide, you'll learn:

* What a Linux cron job is
* How cron jobs are scheduled
* The difference between user and system cron jobs
* Why cron jobs are important during privilege escalation
* How the **pspy** tool monitors running processes
* How misconfigured cron jobs can lead to root access

> **Note:** The techniques discussed here are for cybersecurity education, penetration testing, and defensive security assessments. Only perform them on systems you own or are authorized to test.

---

# What is a Cron Job?

A **cron job** is a task that Linux executes **automatically at scheduled times**.

Instead of manually running a command every day or every hour, Linux can execute it automatically.

Think of cron as the Linux equivalent of a scheduler or task planner.

```text
Specific Time

        ↓

Cron Service

        ↓

Runs Command or Script
```

---

# Why are Cron Jobs Used?

Cron jobs are commonly used for repetitive tasks.

Examples include:

* Creating backups
* Cleaning temporary files
* Restarting services
* Sending reports
* Renewing SSL/TLS certificates
* Running maintenance scripts

For example, a company might automatically back up its database every night at 2:00 AM.

---

# Why are Cron Jobs Important for Security?

During privilege escalation, cron jobs are one of the first things you should inspect.

Why?

Because a poorly configured cron job might:

* Execute a writable script
* Run as the **root** user
* Execute files from writable directories
* Use insecure permissions

If an attacker can modify what the cron job executes, they may be able to run their own code with elevated privileges.

---

# User Cron Jobs

Every Linux user can have their own scheduled tasks.

These are managed using the **crontab** command.

## View Your Cron Jobs

```bash
crontab -l
```

This lists all cron jobs configured for the current user.

Example:

```text
0 2 * * * /home/alice/backup.sh
```

This means:

> Run `backup.sh` every day at **2:00 AM**.

---

## Edit Your Cron Jobs

```bash
crontab -e
```

This opens the user's crontab file in a text editor.

You can:

* Add new jobs
* Edit existing jobs
* Remove jobs

Each user has their own separate crontab.

---

# Understanding Cron Schedule Fields

Every cron job starts with **five scheduling fields**.

```text
Minute Hour Day Month Day_of_Week
```

| Field        | Values                         |
| ------------ | ------------------------------ |
| Minute       | 0–59                           |
| Hour         | 0–23                           |
| Day of Month | 1–31                           |
| Month        | 1–12                           |
| Day of Week  | 0–7 (Sunday is usually 0 or 7) |

---

## Example

```text
0 9 15 2 *
```

This means:

* Minute → `0`
* Hour → `9`
* Day → `15`
* Month → `2` (February)
* Day of Week → `*` (any day)

Result:

> Execute the command at **9:00 AM on February 15th**.

---

# What Does `*` Mean?

The asterisk (`*`) is called a **wildcard**.

It means:

> **"Any value."**

Example:

```text
* * * * *
```

Meaning:

> Run the command **every minute**.

Another example:

```text
0 * * * *
```

Meaning:

> Run the command at the start of **every hour**.

---

# System-Wide Cron Jobs

Some scheduled tasks apply to the entire operating system rather than a single user.

These are configured in:

```text
/etc/crontab
```

Unlike user crontabs, this file must specify **which user** should execute each command.

Example:

```text
0 2 * * * root /usr/local/bin/backup.sh
```

Here:

* Time → 2:00 AM
* User → root
* Command → `/usr/local/bin/backup.sh`

The username is required because this file schedules jobs for multiple users.

---

# Cron Directories

Linux also provides special directories for scheduled tasks.

```text
/etc/cron.hourly
/etc/cron.daily
/etc/cron.weekly
/etc/cron.monthly
/etc/cron.d
```

These directories automatically execute scripts at specific intervals.

For example:

```text
/etc/cron.daily
```

Every executable script inside this directory runs **once every day**.

Unlike a crontab, you do **not** need to specify scheduling fields—the directory itself determines how often the scripts are executed.

---

# Why Should You Enumerate Cron Jobs?

Cron jobs often reveal hidden privilege escalation opportunities.

During enumeration, ask questions like:

* Which cron jobs are running?
* Which user executes them?
* Which scripts do they run?
* Can I modify those scripts?
* Can I write files into the directories they use?

If the answer to the last two questions is **yes**, privilege escalation may be possible.

---

# Process Monitoring with `pspy`

## What is pspy?

`pspy` is a Linux tool that allows you to monitor running processes **without root privileges**.

It is especially useful for discovering:

* Background scripts
* Cron jobs
* Short-lived processes
* Automated tasks

Many cron jobs execute so quickly that you might never notice them using normal monitoring tools.

`pspy` helps reveal these hidden processes.

---

# How Does pspy Work?

Linux stores information about every running process inside the:

```text
/proc
```

directory.

Each running process has its own folder.

Example:

```text
/proc/1234
/proc/5678
/proc/9102
```

Each folder contains information such as:

* Process name
* Command line
* Memory usage
* Process status

For example:

```text
/proc/<PID>/cmdline
```

contains the command that started the process.

---

# Detecting New Processes

`pspy` continuously watches the `/proc` filesystem.

Whenever a new process starts, it immediately displays information such as:

* Process ID (PID)
* Parent process
* User
* Command being executed

Because it watches the system in real time, it can detect very short-lived cron jobs that may only run for a few seconds.

---

# Why Doesn't pspy Need Root?

Normally, monitoring processes requires elevated privileges.

However, `pspy` relies on information that Linux already exposes through the `/proc` filesystem.

It also uses a Linux feature called **inotify**, which notifies programs whenever files or directories change.

This allows `pspy` to monitor process activity efficiently without placing a heavy load on the system.

---

# Static Binaries

`pspy` is usually distributed as a **statically linked binary**.

This means:

* All required libraries are included inside the executable.
* It does not depend on external system libraries.
* It can run on many Linux systems without installation.

This makes it very useful during penetration tests, where installing software on the target system is often not possible.

---

# Example Privilege Escalation Scenario

The following example demonstrates how a misconfigured cron job can lead to root access.

## Step 1 – Initial Access

The attacker discovers a PHP web shell inside a web application's `/dev` directory.

Using this web shell, they obtain a basic reverse shell on the target machine.

```text
Web Shell

↓

Reverse Shell

↓

Limited User Access
```

---

## Step 2 – Pivot to Another User

During enumeration, the attacker discovers that the current user can switch to another account named:

```text
scriptmanager
```

using a permissive `sudo` configuration.

Now the attacker has access as the `scriptmanager` user.

---

## Step 3 – Monitor Running Processes

The attacker runs:

```text
pspy
```

After watching the output for a short time, they notice a recurring process similar to:

```text
root

↓

cd /scripts

↓

Execute every .py file
```

This means the **root** user periodically executes Python scripts located inside the `/scripts` directory.

---

## Step 4 – Check Permissions

Next, the attacker checks the directory permissions.

They discover:

```text
/scripts

↓

Writable by scriptmanager
```

This is a serious security misconfiguration.

Because `scriptmanager` can write files there, they control what the root cron job executes.

---

## Step 5 – Exploitation

The attacker places a Python script into the directory.

When the cron job runs again:

```text
Root Cron Job

↓

Executes Attacker's Python Script

↓

Script Runs as Root

↓

Root Access
```

The attacker successfully escalates privileges.

---

# How to Secure Cron Jobs

To reduce the risk of privilege escalation:

* Do not allow untrusted users to modify scripts executed by cron.
* Restrict write permissions on cron directories and scripts.
* Regularly review cron jobs for unnecessary root privileges.
* Use absolute file paths in cron jobs.
* Remove unused or outdated scheduled tasks.
* Monitor changes to cron configurations and executable scripts.

---

# Key Takeaways

* A **cron job** is a scheduled task that runs automatically in Linux.
* Users manage their own cron jobs using `crontab -l` and `crontab -e`.
* System-wide cron jobs are stored in `/etc/crontab` and the `/etc/cron.*` directories.
* The five cron schedule fields define **when** a task runs.
* Misconfigured cron jobs are a common source of privilege escalation.
* `pspy` is a powerful process monitoring tool that can detect short-lived background processes without requiring root access.
* Always check **who** runs a cron job, **what** it executes, and **whether you can modify** the files or directories involved.
