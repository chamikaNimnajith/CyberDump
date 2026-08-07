# DLL Hijacking 

---

# 1. What is a DLL?

A **Dynamic Link Library (DLL)** is a file that contains code and functions that multiple programs can share.

Instead of every program having its own copy of common code, Windows stores that code in a DLL. This saves **disk space** and **memory**.

### Example

Many applications use the same Windows functions.

Instead of including those functions in every program:

```
Program A  ──► Windows DLL
Program B  ──► Windows DLL
Program C  ──► Windows DLL
```

All programs share the same DLL.

---

# 2. How Does Windows Load a DLL?

When a program starts:

1. Windows reads the executable.
2. It checks which DLLs the program needs.
3. Windows locates those DLL files.
4. The DLLs are loaded into memory.
5. The program calls functions from those DLLs.

This process happens automatically every time the application starts.

---

# 3. Why is DLL Loading a Security Risk?

Because DLLs are loaded **at runtime**, Windows must locate the DLL on the system.

If an attacker can trick Windows into loading a **malicious DLL** instead of the legitimate one, the attacker's code executes with the same privileges as the application.

This attack is known as **DLL Hijacking**.

---

# 4. Two Common DLL Hijacking Techniques

## Technique 1 – DLL Overriding

The attacker replaces the original DLL with a malicious DLL.

Example:

Original application folder:

```
Application
│
├── app.exe
└── lib.dll
```

The attacker replaces **lib.dll** with a malicious version.

When **app.exe** starts, it loads the attacker's DLL instead of the original one.

---

## Technique 2 – DLL Search Order Hijacking

Sometimes an application loads a DLL using only its filename:

```
lib.dll
```

instead of the full path:

```
C:\Program Files\App\lib.dll
```

Windows must search for the DLL.

If the attacker places a malicious DLL in a directory that Windows searches **before** the legitimate DLL, Windows loads the malicious one first.

---

# 5. Example Program

The video demonstrates a simple DLL.

## The DLL (lib.c)

The DLL contains one function:

```
AddNumbers(a, b)
```

It simply adds two numbers and returns the result.

---

## Compiling the DLL

The library is compiled using GCC with the **-shared** option.

Example:

```bash
gcc -shared -o lib.dll lib.c
```

The **-shared** option tells the compiler to create a DLL instead of a normal executable.

---

## The Main Program

The main application:

* Loads **lib.dll**
* Finds the **AddNumbers()** function
* Calls the function
* Prints the result
* Unloads the DLL

---

## Missing DLL

If **main.exe** is copied without **lib.dll**, Windows cannot find the required library.

The program fails to start.

After copying **lib.dll** into the same directory, the program works correctly.

Example output:

```
5 + 10 = 15
```

---

# 6. DLL Overriding Attack

The attacker replaces the legitimate DLL with a malicious DLL.

There are two common approaches.

---

## Method 1 – Stealthy DLL

The malicious DLL exports the **same functions** as the original DLL.

For example:

```
AddNumbers()
```

still exists.

When the program calls the function:

1. The malicious payload runs first.
2. The original calculation is performed.
3. The correct result is returned.

To the user, everything appears normal.

Example payload:

* Create a file named **hacked**
* Write **"hacks"** into the file
* Return the correct addition result

Since the program still behaves correctly, users usually do not notice anything unusual.

### Advantage

* Very difficult to detect.
* Application continues working normally.

### Disadvantage

* The attacker must recreate all exported functions.
* This can be difficult for large applications.

---

## Method 2 – DllMain Hijacking

Every DLL has an entry point called **DllMain**.

Windows automatically executes **DllMain** when the DLL is loaded.

Instead of recreating functions, the attacker places the payload inside **DllMain**.

```
DLL Loaded
      │
      ▼
DllMain()
      │
      ▼
Run malicious code
```

This is much easier to create.

However, the expected functions no longer exist.

The application usually crashes after loading the DLL.

### Advantage

* Easy to create.

### Disadvantage

* The application crashes.
* Users quickly notice something is wrong.

---

# 7. DLL Hijacking in Windows Services

Windows Services are attractive targets because they:

* Run continuously
* Often run with **SYSTEM** privileges
* Can be restarted to trigger DLL loading

If a service loads a malicious DLL, the attacker's code executes with the service's privileges.

---

# 8. Viewing Loaded DLLs

A useful Sysinternals tool is **ListDLLs**.

Command:

```cmd
listdlls64.exe -accept_eula <process_name>
```

Example:

```cmd
listdlls64.exe -accept_eula simpleService
```

The tool displays:

* DLL name
* Memory address
* File size
* File location

This helps identify where a service is loading its DLLs from.

> **Note:** Administrative privileges are required to inspect many processes.

---

# 9. DLL Search Order

If a DLL is loaded without a full path, Windows searches for it in the following order:

1. Process directory
2. System directory (`C:\Windows\System32`)
3. 16-bit System directory
4. Windows directory (`C:\Windows`)
5. Current directory
6. Directories listed in the **PATH** environment variable

Windows loads the **first matching DLL** it finds.

---

# 10. Search Order Hijacking Example

Suppose the application loads:

```
lib.dll
```

The legitimate DLL is stored in:

```
C:\Windows\lib.dll
```

Windows searches:

1. Process directory
2. System32
3. Windows

Now the attacker places a malicious DLL in:

```
C:\Windows\System32\lib.dll
```

Since **System32** is searched **before** the Windows directory, Windows loads the malicious DLL instead.

The malicious code executes before the legitimate DLL is ever reached.

After removing the malicious DLL, Windows loads the original DLL again, and the application works normally.

---

# Detection Tips

Look for applications or services that:

* Load DLLs using **relative paths**.
* Search writable directories.
* Run with **high privileges** (such as SYSTEM).
* Load DLLs from unexpected locations.

Tools such as **Process Monitor (ProcMon)** and **ListDLLs** can help identify DLL loading behavior.

---

# Prevention

To reduce the risk of DLL hijacking:

* Use **absolute DLL paths** whenever possible.
* Store DLLs in secure, access-controlled directories.
* Remove unnecessary write permissions from application folders.
* Digitally sign DLLs when appropriate.
* Regularly monitor DLL loading using security tools.

---

