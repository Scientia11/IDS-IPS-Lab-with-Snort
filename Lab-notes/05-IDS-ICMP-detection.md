## Custom Rule Creation and Alert Mode Testing

# Overview

In this lab, I explored the Intrusion Detection System (IDS) capabilities of Snort by:

* Creating a custom ICMP detection rule
* Studying Snort alert modes
* Running Snort in live alert mode
* Generating test traffic
* Verifying real-time intrusion detection alerts

This lab demonstrated how Snort performs signature-based network intrusion detection using custom rules.

---

# Lab Environment

| Component           | Details             |
| ------------------- | ------------------- |
| Operating System    | Kali Linux          |
| IDS Tool            | Snort               |
| Interface Monitored | `eth0`              |
| Configuration File  | `etc/snort.conf`    |
| Rule File           | `rules/local.rules` |

---

# Step 1 — Creating a Custom Snort Rule

I began by creating my first custom Snort detection rule inside the local rules file.

### File Edited

```bash id="g6f1xm"
rules/local.rules
```

### Rule Created

```snort id="z8m1rd"
alert icmp any any -> 8.8.8.8 any (msg: "ICMP traffic to 8.8.8.8 detected"; sid: 10000001; rev: 1;)
```

---

# Purpose of the Rule

This rule was designed to detect ICMP traffic (such as ping requests) destined for:

```text id="g5w7jk"
8.8.8.8
```

When matching traffic is detected, Snort generates an alert.

---

# Rule Breakdown

| Component     | Meaning                                    |
| ------------- | ------------------------------------------ |
| `alert`       | Generate an alert when traffic matches     |
| `icmp`        | Inspect ICMP protocol traffic              |
| `any any`     | Match any source IP and source port        |
| `->`          | Traffic direction operator                 |
| `8.8.8.8 any` | Destination IP and any port                |
| `msg`         | Alert message displayed when rule triggers |
| `sid`         | Unique Snort rule ID                       |
| `rev`         | Rule revision number                       |

---

# Step 2 — Exploring Snort Manual Pages

I then accessed the Snort manual pages to study Snort operational modes and command-line arguments.

### Command Used

```bash id="0g1n9s"
man snort
```

---

# Focus Area — Alert Modes (`-A`)

I specifically studied the `-A` option, which controls how Snort displays alerts.

General syntax:

```bash id="5n3v7d"
snort -A <mode>
```

---

# Alert Modes Studied

## Fast Alert Mode

```bash id="7m2k0r"
-A fast
```

* Displays concise alerts
* Optimized for quick monitoring

---

## Full Alert Mode

```bash id="r9v4q1"
-A full
```

* Displays detailed packet information
* Useful for deeper analysis

---

## No Alert Mode

```bash id="h4x8m6"
-A none
```

* Disables alert display
* Useful for silent logging

---

## UNIX Socket Alert Mode

```bash id="d6p2z7"
-A unsock
```

* Sends alerts through UNIX sockets
* Useful for external integrations and SIEM systems

---

# Step 3 — Running Snort in Alert Mode

After understanding the alert modes, I launched Snort in live console alert mode.

### Command Used

```bash id="v1q4k9"
sudo snort -A console -l ~/log -i eth0 -c etc/snort.conf -q
```

---

# Command Breakdown

| Option              | Purpose                                 |
| ------------------- | --------------------------------------- |
| `-A console`        | Display alerts directly on the terminal |
| `-l ~/log`          | Store logs inside the `~/log` directory |
| `-i eth0`           | Monitor traffic on interface `eth0`     |
| `-c etc/snort.conf` | Use specified Snort configuration file  |
| `-q`                | Run Snort in quiet mode                 |

---

# Step 4 — Testing the Rule

## First Test — Ping to 8.8.4.4

I first generated ICMP traffic by pinging:

```text id="4y5r1n"
8.8.4.4
```

### Result

No alert was generated.

### Reason

The rule specifically targeted traffic destined for:

```text id="6x0k2d"
8.8.8.8
```

This confirmed that Snort rules only trigger when traffic exactly matches the defined rule conditions.

---

# Second Test — Ping to 8.8.8.8

I then generated ICMP traffic by pinging:

```text id="v7n4e2"
8.8.8.8
```

### Result

Snort immediately generated live alerts on the terminal.

Example alert:

```text id="m8p2w5"
05/08-01:31:47.675737  [**] [1:10000001:1] ICMP traffic to 8.8.8.8 detected [**] [Priority: 0] {ICMP} 10.0.2.15 -> 8.8.8.8
```

---

# Alert Breakdown

| Field                  | Meaning                                |
| ---------------------- | -------------------------------------- |
| Timestamp              | Time the event occurred                |
| `[1:10000001:1]`       | Generator ID : Signature ID : Revision |
| Alert Message          | Custom rule message                    |
| `{ICMP}`               | Protocol detected                      |
| `10.0.2.15 -> 8.8.8.8` | Source IP → Destination IP             |

---

# Key Concepts Learned

During this lab, I learned:

* How to create custom Snort rules
* Snort rule syntax structure
* How IDS alerting works
* How to run Snort in live alert mode
* How Snort matches packets against signatures
* How ICMP traffic can be monitored and detected
* How alert modes affect Snort output behavior

---

# Skills Practiced

* IDS configuration
* Signature-based detection
* Snort rule writing
* ICMP traffic analysis
* Live traffic monitoring
* Snort alert analysis
* Network packet inspection

---

# Important Observation

This lab demonstrated the core concept of signature-based intrusion detection:

* Traffic that does not match the rule is ignored
* Traffic matching the rule generates alerts immediately

The experiment also showed how each ICMP echo request generated a separate detection event.

---

# Conclusion

In this lab, I successfully:

* Created a custom Snort detection rule
* Explored Snort alert modes
* Ran Snort in live IDS alert mode
* Generated test ICMP traffic
* Validated real-time intrusion detection alerts

This exercise strengthened my understanding of how Snort performs signature-based intrusion detection and real-time traffic monitoring in a SOC environment.

