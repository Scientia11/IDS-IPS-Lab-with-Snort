# Snort 2 Logger Mode Lab

## Overview

In this lab, I explored Snort 2 Logger Mode to understand how Snort captures, stores, and replays network traffic for offline analysis.

The objective of this lab was to:

* Capture live packets
* Store packets into log files
* Read captured traffic using Snort
* Analyze the same log files using `tcpdump`
* Understand how packet logging works in a Network Intrusion Detection System (NIDS)

Environment used:

* Operating System: Kali Linux
* IDS Tool: Snort
* Packet Analyzer: tcpdump

---

# Step 1 — Creating a Log Directory

Before logging packets, I created a dedicated directory to store Snort log files.

### Commands Used

```bash id="q0ehpn"
mkdir log
```

```bash id="jx6w1s"
cd log
```

```bash id="4j7b2q"
cd ~/snort-2.9.20/etc
```

### Purpose

This directory was created to:

* Organize packet capture logs
* Separate Snort logs from other files
* Make packet analysis easier

---

# Step 2 — Running Snort in Logger Mode

I then started Snort in packet logging mode.

### Command Used

```bash id="qzq2hc"
sudo snort -i eth0 -l ~/log
```

---

# Understanding the Command

| Option     | Meaning                                        |
| ---------- | ---------------------------------------------- |
| `-i eth0`  | Capture traffic from the `eth0` interface      |
| `-l ~/log` | Store packet logs inside the `~/log` directory |

---

# Result

Snort successfully started in packet logging mode.

Example output:

```text id="e6r3e4"
Running in packet logging mode
Log directory = /home/godsway/log
Acquiring network traffic from "eth0".
```

Snort began capturing and storing live packets into log files.

Generated log files were automatically named in the format:

```text id="4l7q63"
snort.log.<timestamp>
```

Example:

```text id="4zqf1s"
snort.log.1778137269
```

---

# Step 3 — Reading Logged Packets with Snort

After generating packet logs, I replayed the stored packets using Snort’s read mode.

### Command Used

```bash id="8qzv0m"
sudo snort -r ~/log/snort.log.1778137269
```

---

# Purpose

This command was used to:

* Read previously captured traffic
* Replay packets offline
* Analyze network activity without needing live traffic
* Verify successful packet logging

---

# Result

Snort successfully opened the log file and displayed captured packets.

Observed traffic included:

* TCP packets
* UDP packets
* DNS traffic
* Source and destination IP addresses
* Sequence and acknowledgment numbers

Example observed traffic:

```text id="jv4j8x"
10.0.2.15 -> 104.18.39.21:443
104.18.39.21 -> 10.0.2.15:48022
10.0.2.15 -> 8.8.8.8:53
```

---

# Step 4 — Reading Snort Log Files with tcpdump

I then verified the Snort-generated log file using `tcpdump`.

### Command Used

```bash id="6b0t5j"
sudo tcpdump -r ~/log/snort.log.1778137269
```

---

# Purpose

This step demonstrated that:

* Snort log files are compatible with other packet analysis tools
* Packet captures can be analyzed outside Snort
* Logged traffic can be replayed and inspected using multiple tools

---

# Result

`tcpdump` successfully parsed the Snort log file.

Example output:

```text id="a8m7rw"
03:01:21.956578 IP 10.0.2.15.60968 > dns.google.domain: 58348+ A? example.com. (29)
```

This showed a DNS query for `example.com`.

DNS response observed:

```text id="37n7uq"
03:01:22.344996 IP dns.google.domain > 10.0.2.15.60968: 58348 2/0/0 A 104.20.23.154, A 172.66.147.243 (61)
```

This confirmed successful DNS resolution.

---

# Key Observations

During the lab, I observed:

* DNS queries and responses
* TCP acknowledgments
* HTTPS traffic
* IPv4 and IPv6 packets
* Packet timestamps
* TCP flags
* Network session activity

---

# Skills Practiced

This lab helped me practice:

* Packet logging with Snort
* Offline packet analysis
* Network traffic inspection
* DNS traffic analysis
* Reading packet captures
* Using multiple packet analysis tools
* Understanding TCP/IP communication

---

# Important Commands Summary

| Task                             | Command                                      |
| -------------------------------- | -------------------------------------------- |
| Create log directory             | `mkdir log`                                  |
| Run Snort in logger mode         | `sudo snort -i eth0 -l ~/log`                |
| Read logged packets with Snort   | `sudo snort -r ~/log/snort.log.1778137269`   |
| Read logged packets with tcpdump | `sudo tcpdump -r ~/log/snort.log.1778137269` |

---

# Conclusion

In this lab, I successfully configured and used Snort 2 Logger Mode to:

* Capture live network traffic
* Store packets into log files
* Replay captured packets offline
* Analyze packet captures using both Snort and `tcpdump`

This exercise improved my understanding of:

* Packet logging workflows
* Offline packet analysis
* Network traffic inspection
* IDS traffic monitoring techniques

The lab also demonstrated how Snort integrates with other packet analysis tools for deeper traffic inspection and troubleshooting.

