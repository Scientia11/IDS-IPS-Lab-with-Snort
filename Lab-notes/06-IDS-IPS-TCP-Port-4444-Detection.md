Copy this into today’s GitHub lab note:


# IDS/IPS with Snort — TCP Port 4444 Detection Using hping3

## Overview

In this lab, I created and tested a custom Snort rule to detect TCP traffic going to destination port `4444`.

The purpose of this lab was to understand how Snort detects suspicious TCP traffic using custom rules and how `hping3` can be used to generate controlled test packets.

---

## Lab Environment

| Component | Details |
|---|---|
| Operating System | Kali Linux |
| IDS Tool | Snort 2.9.20 |
| Packet Generator | hping3 |
| Interface | eth0 |
| Rule File | rules/local.rules |
| Config File | etc/snort.conf |
| Log Directory | ~/log |

---

## Step 1 — Disabled Previous ICMP Rule

I first commented out the old ICMP rule created in the previous lab. This allowed me to focus only on the new TCP port-based detection rule.

---

## Step 2 — Created TCP Port 4444 Detection Rule

I added the following rule to `rules/local.rules`:

```snort
alert tcp any any -> any 4444 (msg: "Connection to Remote IP on Port 4444 detected"; sid: 10000002; rev: 1;)
````

### Rule Breakdown

| Rule Part      | Meaning                                     |
| -------------- | ------------------------------------------- |
| `alert`        | Generate an alert                           |
| `tcp`          | Inspect TCP traffic                         |
| `any any`      | Any source IP and source port               |
| `->`           | Direction of traffic                        |
| `any 4444`     | Any destination IP on destination port 4444 |
| `msg`          | Alert message                               |
| `sid:10000002` | Unique rule ID                              |
| `rev:1`        | Rule revision                               |

---

## Step 3 — Started Snort in Alert Mode

I ran Snort in console alert mode:

```bash
sudo snort -A console -l ~/log -i eth0 -c etc/snort.conf -q
```

### Command Breakdown

| Option              | Meaning                          |
| ------------------- | -------------------------------- |
| `-A console`        | Display alerts on the terminal   |
| `-l ~/log`          | Save logs in the log directory   |
| `-i eth0`           | Monitor the eth0 interface       |
| `-c etc/snort.conf` | Use the Snort configuration file |
| `-q`                | Run in quiet mode                |

---

## Step 4 — Sent Test Traffic to Port 4455

I first sent a TCP SYN packet to port `4455`:

```bash
sudo hping3 -c 1 -p 4455 -S example.com
```

### Result

Snort did not generate an alert.

### Reason

The rule was designed to detect traffic going to port `4444`, not port `4455`.

---

## Step 5 — Sent Test Traffic to Port 4444

I then sent four TCP SYN packets to port `4444`:

```bash
sudo hping3 -c 4 -p 4444 -S example.com
```

### Result

Snort generated alerts for the traffic.

Observed alert output:

```text
05/09-05:07:29.591372  [**] [1:10000002:1] Connection to Remote IP on Port 4444 detected [**] [Priority: 0] {TCP} 10.0.2.15:2841 -> 172.66.147.243:4444
05/09-05:07:30.591952  [**] [1:10000002:1] Connection to Remote IP on Port 4444 detected [**] [Priority: 0] {TCP} 10.0.2.15:2842 -> 172.66.147.243:4444
05/09-05:07:31.592998  [**] [1:10000002:1] Connection to Remote IP on Port 4444 detected [**] [Priority: 0] {TCP} 10.0.2.15:2843 -> 172.66.147.243:4444
05/09-05:07:32.593678  [**] [1:10000002:1] Connection to Remote IP on Port 4444 detected [**] [Priority: 0] {TCP} 10.0.2.15:2844 -> 172.66.147.243:4444
```

---

## Alert Breakdown

| Field                                           | Meaning                                |
| ----------------------------------------------- | -------------------------------------- |
| `05/09-05:07:29.591372`                         | Timestamp                              |
| `[1:10000002:1]`                                | Generator ID : Signature ID : Revision |
| `Connection to Remote IP on Port 4444 detected` | Custom alert message                   |
| `{TCP}`                                         | Protocol detected                      |
| `10.0.2.15:2841`                                | Source IP and source port              |
| `172.66.147.243:4444`                           | Destination IP and destination port    |

---

## Key Observation

Snort did not alert when traffic was sent to port `4455`, but it alerted when traffic was sent to port `4444`.

This confirmed that the rule was working as intended.

Because I used:

```bash
-c 4
```

Snort generated four alerts, one for each TCP SYN packet.

---

## Skills Practiced

* Writing Snort TCP detection rules
* Commenting out old rules
* Running Snort in alert mode
* Testing detection rules with hping3
* Detecting TCP SYN traffic
* Understanding port-based IDS detection
* Analyzing Snort alert output

---

## Conclusion

This lab demonstrated how Snort can detect TCP traffic based on destination ports.

I successfully created a custom rule to detect traffic going to port `4444`, tested it with `hping3`, and confirmed that Snort generated alerts only when the traffic matched the rule conditions.

```
