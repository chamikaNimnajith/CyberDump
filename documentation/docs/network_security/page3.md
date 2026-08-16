# Intrusion Detection and Prevention Systems

Modern networks continuously exchange large amounts of traffic between users, applications, servers, and external systems. Among this traffic, some connections may be legitimate while others may represent scanning, exploitation attempts, malware communication, unauthorized access, or other malicious activity.

**Intrusion Detection Systems (IDS)** and **Intrusion Prevention Systems (IPS)** are security technologies designed to identify suspicious activity and help organizations respond to network-based threats.

---

## 1. Basic Overview of IDS and IPS

### What Is an IDS?

An **Intrusion Detection System (IDS)** monitors network traffic or system activity and looks for signs of malicious or suspicious behavior.

When the IDS identifies a potential threat, it generates an **alert** so that a security administrator or another security system can investigate it.

A simplified workflow is:

```text
Network Traffic
      ↓
     IDS
      ↓
Analyze activity
      ↓
Threat detected?
      ↓
    Alert
      ↓
Security Administrator
```

An IDS is therefore primarily a **detection and alerting mechanism**.

It can help security teams identify:

* Port scanning
* Suspicious connections
* Malware traffic
* Exploitation attempts
* Policy violations
* Unusual network behavior

An IDS generally does not directly block the connection that generated the alert.

---

### What Is an IPS?

An **Intrusion Prevention System (IPS)** performs detection similar to an IDS but can also take **automated action** when malicious activity is identified.

For example, an IPS may:

* Drop malicious packets
* Block a connection
* Reject a request
* Temporarily block an IP address
* Prevent further communication with a suspected attacker

The workflow can therefore be represented as:

```text
Network Traffic
      ↓
     IPS
      ↓
Analyze activity
      ↓
Threat detected?
      ↓
Block / Drop / Reject
      ↓
Prevent further activity
```

The key difference is:

> **IDS detects and alerts, while IPS detects and can automatically prevent.**

---

## 2. Detection Methodologies

IDS and IPS technologies can use different techniques to determine whether network activity is suspicious.

Two important approaches are:

1. **Signature-based detection**
2. **Anomaly-based detection**

---

## 2.1 Signature-Based Detection

**Signature-based detection** identifies threats by comparing observed activity against a database of known attack signatures.

A **signature** is a recognizable pattern associated with a particular attack, malware family, exploit, or malicious activity.

The general process is:

```text
Network Activity
      ↓
Extract characteristics
      ↓
Compare with signatures
      ↓
Match found?
   ↙       ↘
 Yes        No
 ↓           ↓
Alert /     Continue
Block       monitoring
```

For example, a security system may contain a signature that identifies a known malicious packet sequence.

When incoming traffic matches that pattern, the system can generate an alert or block the traffic.

### Examples of Indicators

Signature-based systems may look for indicators such as:

* Known malicious file hashes
* Connections to known malicious domains
* Specific malicious byte sequences
* Known exploit patterns
* Recognizable phishing-related email characteristics
* Known malware communication patterns

For example:

```text
Known malicious hash
        ↓
File hash comparison
        ↓
Match
        ↓
Potential malware detected
```

### Advantages

Signature-based detection is generally effective at identifying **known threats** because the system already has a description of what to look for.

It can also produce relatively precise detections when signatures are well designed.

### Limitation

Its major limitation is that it depends on **known signatures**.

If an attacker creates a completely new attack that does not match an existing signature, signature-based detection may fail to identify it.

This is one reason anomaly-based detection is also important.

---

# 2.2 Anomaly-Based Detection

**Anomaly-based detection** attempts to identify activity that deviates from what is considered normal behavior.

Instead of asking:

> "Does this traffic match a known attack?"

the system asks:

> "Does this activity look unusual compared with the normal baseline?"

A simplified process is:

```text
Normal Activity
      ↓
Establish baseline
      ↓
Monitor current activity
      ↓
Compare with baseline
      ↓
Significant deviation?
      ↓
Potential anomaly
```

For example, suppose an employee normally logs into a system between 8:00 AM and 5:00 PM.

If the same account suddenly logs in at 3:00 AM from an unusual location, the activity may be considered anomalous.

### Examples of Anomalous Behavior

Possible anomalies include:

* Login activity outside normal working hours
* Unusually large increases in network traffic
* An employee suddenly transferring a large amount of data
* An unauthorized device appearing on the network
* A server communicating with an unusual destination
* A user accessing resources they do not normally use

For example:

```text
Normal:
User → 20 MB/day

Observed:
User → 8 GB/day

       ↓
Large deviation
       ↓
Potential anomaly
```

### Advantage

Anomaly-based detection can be useful for identifying **previously unknown or new threats**, because it does not necessarily require a predefined signature for the attack.

### Challenge

The main challenge is determining what is genuinely abnormal.

For example, a legitimate backup operation may suddenly generate a large amount of network traffic. An anomaly detection system might initially consider this suspicious even though it is legitimate.

This can result in **false positives**.

---

# 3. IDS and IPS Comparison

IDS and IPS systems share many characteristics because both are designed to identify suspicious activity.

However, their response mechanisms are different.

## Similarities

Both IDS and IPS can perform continuous monitoring of:

* Network traffic
* Network connections
* Hosts and devices
* Protocol activity
* User behavior
* System events

They can also generate alerts when suspicious activity is identified.

A typical security monitoring process is:

```text
              Network / Host Activity
                       ↓
                    Monitor
                       ↓
                    Analyze
                       ↓
                 Threat detected
                       ↓
                    Alert
                       ↓
                  Record event
```

Modern systems may also use historical information and feedback to improve detection and reduce false positives.

Both systems commonly maintain **logs** containing information about detected events and system activity.

These logs can be useful for:

* Security investigations
* Incident response
* Troubleshooting
* Performance analysis
* Compliance
* Forensic investigations

---

## Differences Between IDS and IPS

| Feature                               | IDS                              | IPS                        |
| ------------------------------------- | -------------------------------- | -------------------------- |
| Primary purpose                       | Detect threats                   | Detect and prevent threats |
| Monitoring                            | Yes                              | Yes                        |
| Alerting                              | Yes                              | Yes                        |
| Automatic blocking                    | Normally no                      | Yes                        |
| Response                              | Alerts administrators            | Takes automated action     |
| Network position                      | Often monitors traffic passively | Usually positioned inline  |
| Risk of disrupting legitimate traffic | Lower                            | Higher                     |
| Immediate protection                  | Limited                          | Higher                     |

### IDS Example

Suppose an attacker performs a port scan:

```text
Attacker
   ↓
Port scanning
   ↓
IDS
   ↓
Detects scanning
   ↓
Alert generated
```

The scan may continue, but the security team is informed.

### IPS Example

With an IPS:

```text
Attacker
   ↓
Port scanning
   ↓
IPS
   ↓
Detects scanning
   ↓
Blocks traffic
```

The IPS attempts to stop the suspicious activity automatically.

---

## Operational Impact of IPS

The ability of an IPS to automatically block traffic provides stronger immediate protection, but it also introduces additional risk.

Suppose an IPS incorrectly identifies legitimate traffic as malicious:

```text
Legitimate User
      ↓
Normal request
      ↓
IPS
      ↓
False positive
      ↓
Request blocked
```

If the IPS is configured too aggressively, it could:

* Block legitimate users
* Interrupt applications
* Prevent valid network connections
* Disconnect devices
* Cause service availability problems

In extreme situations, incorrect prevention rules can cause significant network disruption.

Therefore, IPS systems require careful configuration, testing, monitoring, and tuning.

---

# 4. SNORT

**Snort** is a widely used open-source network intrusion detection and prevention technology.

It can analyze network traffic and use rules to identify suspicious patterns.

A Snort rule essentially tells the system:

> **When traffic matches this condition, perform this action.**

The general structure is:

```text
action protocol source direction destination (options)
```

For example:

```text
alert tcp any any -> any 80 (msg:"HTTP traffic";)
```

This rule can be interpreted as:

```text
Action      → alert
Protocol    → TCP
Source      → any IP, any port
Direction   → ->
Destination → any IP, port 80
```

The options inside parentheses provide additional conditions and information.

---

# 5. Snort Rule Actions

Snort provides several rule actions.

### `alert`

Generates an alert and normally logs information about the detected activity.

```text
alert
```

This is one of the most commonly used actions when creating detection rules.

---

### `log`

Records matching traffic without generating a standard alert.

```text
log
```

This can be useful when traffic needs to be recorded for monitoring or analysis.

---

### `pass`

Tells Snort to ignore matching traffic.

```text
pass
```

This can be useful for excluding known legitimate traffic from further inspection.

---

### `activate`

An `activate` rule generates an alert and can activate another rule, such as a `dynamic` rule.

```text
activate
```

This allows multiple rules to work together.

---

### `dynamic`

A `dynamic` rule remains inactive until it is activated by an associated `activate` rule.

```text
dynamic
```

This mechanism can be useful when a particular event should trigger additional inspection.

The relationship can be represented as:

```text
Traffic
   ↓
Activate Rule
   ↓
Alert
   ↓
Dynamic Rule activated
   ↓
Additional traffic inspection
```

---

# 6. Network Variables

Snort rules can use variables to make rules easier to manage.

For example, a variable can represent a collection of internal networks:

```text
HOME_NET
```

Instead of repeatedly writing IP addresses, a rule can refer to the variable.

Conceptually:

```text
HOME_NET = Internal network
```

A rule can then use:

```text
$HOME_NET
```

This makes rules more readable and easier to maintain when the network configuration changes.

---

# 7. TCP Flags

Snort rules can also inspect **TCP flags**.

TCP flags provide information about the state or purpose of TCP segments.

Important flags include:

| Flag  | Meaning |
| ----- | ------- |
| **S** | SYN     |
| **A** | ACK     |
| **F** | FIN     |
| **P** | PSH     |
| **U** | URG     |

### SYN — `S`

The **SYN** flag is commonly associated with initiating a TCP connection.

For example:

```text
Client
  ↓
SYN
  ↓
Server
```

This is part of the TCP three-way handshake.

### ACK — `A`

The **ACK** flag indicates acknowledgment information.

### FIN — `F`

The **FIN** flag is used when a TCP connection is being gracefully terminated.

### PSH — `P`

The **PSH** flag requests that buffered data be pushed to the receiving application.

### URG — `U`

The **URG** flag indicates that urgent data is present.

Snort can also inspect combinations of flags.

For example, a rule may look for a particular combination such as:

```text
S + F
```

Unusual TCP flag combinations can sometimes be associated with network scanning or other reconnaissance techniques.

---

# 8. Port Specifications

Snort rules can specify source and destination ports.

A single port can be specified directly:

```text
80
```

This represents port 80.

Ranges can also be used.

### Range: `1:1024`

```text
1:1024
```

Means:

```text
Ports 1 through 1024
```

### Range: `:6000`

```text
:6000
```

Represents ports from the beginning of the valid range through port 6000.

### Range: `500:`

```text
500:
```

Represents port 500 and higher.

Port ranges allow rules to match broader groups of services without listing every port individually.

---

# 9. Content Matching

One of the most powerful features of Snort rules is the ability to inspect the **content of packets**.

For example:

```text
content:"password";
```

This tells Snort to look for a particular byte sequence or text pattern in the inspected traffic.

Content matching can be further controlled using modifiers.

---

## 9.1 `offset`

The **`offset`** modifier specifies where Snort should begin searching within the packet.

Conceptually:

```text
Packet
0 1 2 3 4 5 6 7 8 9 ...
        ↑
      offset
```

For example:

```text
content:"ABC"; offset:10;
```

The search begins at the specified position rather than at the beginning of the packet.

---

## 9.2 `depth`

The **`depth`** modifier limits how far into the packet Snort should search.

For example:

```text
content:"ABC"; depth:20;
```

This limits the search to the specified portion of the packet.

Using `offset` and `depth` together allows a rule to search a specific region:

```text
offset → Where to start
depth  → How far to search
```

---

## 9.3 `nocase`

The **`nocase`** modifier makes content matching case-insensitive.

For example:

```text
content:"admin"; nocase;
```

This can match variations such as:

```text
admin
Admin
ADMIN
AdMiN
```

Without `nocase`, the exact case of the content may matter.

---

## 9.4 `distance`

The **`distance`** modifier controls how far Snort should move from the previous content match before searching for the next content pattern.

It is useful when a rule needs to identify multiple pieces of content that occur in a particular relative position.

Conceptually:

```text
Pattern A
    ↓
   gap
    ↓
Pattern B
```

`distance` helps define the expected gap between the two patterns.

---

## 9.5 `within`

The **`within`** modifier specifies the maximum number of bytes within which the next content pattern must be found.

For example:

```text
Pattern A
    ↓
    └────── Pattern B
       within specified range
```

This allows a rule to require related content to appear relatively close together.

### `distance` vs `within`

These modifiers are often used together:

```text
distance → How far from the previous match to begin searching
within   → How large the allowed search area is
```

This provides more precise control over packet-content inspection.

---

# 10. Combining Snort Rule Components

A Snort rule can combine multiple conditions to describe a specific type of traffic.

Conceptually:

```text
Action
  +
Protocol
  +
Source
  +
Direction
  +
Destination
  +
TCP flags / ports
  +
Content
  +
Content modifiers
```

For example, a rule might conceptually express:

```text
Alert
  when
TCP traffic
  from an external network
  to an internal server
  on a particular port
  contains a specific byte sequence
  within a defined packet region
```

This allows Snort rules to move beyond simple port-based detection and inspect the actual characteristics of network traffic.

---

# 11. Putting IDS, IPS, and Snort Together

IDS and IPS provide the broader security function of **detecting and responding to suspicious activity**, while rule-based technologies such as Snort provide mechanisms for identifying specific traffic patterns.

The relationship can be viewed as:

```text
                 Network Traffic
                       ↓
                ┌──────────────┐
                │ IDS / IPS    │
                └──────┬───────┘
                       ↓
                  Traffic Analysis
                       ↓
              ┌────────┴────────┐
              │                 │
        Signature Rules     Anomaly Detection
              │                 │
              └────────┬────────┘
                       ↓
                Suspicious Activity
                       ↓
              ┌────────┴────────┐
              │                 │
             IDS               IPS
              │                 │
            Alert          Block / Prevent
```

Understanding IDS and IPS therefore requires looking at both **how suspicious activity is detected** and **what happens after detection**. Signature-based techniques are particularly useful for recognizing known threats, while anomaly-based techniques can identify unusual behavior that may not match previously known attack patterns. Rule-based systems such as Snort provide a practical way to define detailed conditions for inspecting network traffic.
