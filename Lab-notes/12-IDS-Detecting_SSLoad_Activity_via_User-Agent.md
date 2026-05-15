## Snort Rule: Detecting SSLoad Activity via User-Agent

## Objective

The goal of this lab was to write a Snort rule to detect potential SSLoad activity based on a suspicious HTTP User-Agent value found during a previous Wireshark network traffic analysis lab.

In the earlier analysis, I observed HTTP traffic containing the User-Agent:

```text
SSLoad/1.1
````

This User-Agent was used as an indicator for building a custom Snort detection rule.

---

## Step 1: Cleared Old Snort Logs

Before running the new rule, I cleared the existing Snort log directory.

```bash
cd ~/log
sudo rm *
ls
```

This ensured that only logs from the current SSLoad detection test would remain.

---

## Step 2: Reviewed the PCAP in Wireshark

I opened the PCAP file in Wireshark for additional inspection.

```bash
sudo wireshark -r ~/Desktop/02_Network_Security/03_Snort/PCAPs/2.pcap
```

A warning appeared:

```text
Warning: program compiled against libxml 215 using older 214
```

This warning did not prevent Wireshark from opening the PCAP.

---

## Step 3: Created the Snort Rule

I edited the local Snort rules file:

```bash
sudo nano rules/local.rules
```

I added the following rule:

```snort
alert tcp any any -> any any (msg:"Potential SSLoad activity via User-Agent"; content:"User-Agent|3A| SSLoad/1.1"; http_header; nocase; sid:10000007; rev:1;)
```

---

## Rule Breakdown

| Component            | Meaning                                          |              |                                             |
| -------------------- | ------------------------------------------------ | ------------ | ------------------------------------------- |
| `alert`              | Generate an alert when traffic matches           |              |                                             |
| `tcp`                | Inspect TCP traffic                              |              |                                             |
| `any any -> any any` | Match traffic from any source to any destination |              |                                             |
| `msg`                | Alert message displayed when the rule triggers   |              |                                             |
| `content:"User-Agent | 3A                                               | SSLoad/1.1"` | Search for `User-Agent: SSLoad/1.1`         |
| `                    | 3A                                               | `            | Hexadecimal representation of the colon `:` |
| `http_header`        | Restrict inspection to HTTP headers              |              |                                             |
| `nocase`             | Match regardless of letter case                  |              |                                             |
| `sid:10000007`       | Custom Snort rule ID                             |              |                                             |
| `rev:1`              | Rule revision number                             |              |                                             |

---

## Step 4: Ran Snort Against the PCAP

I ran Snort against the PCAP file using offline analysis mode.

```bash
sudo snort -c etc/snort.conf -r ~/Desktop/02_Network_Security/03_Snort/PCAPs/2.pcap -l ~/log -q -A console
```

---

## Alerts Generated

Snort successfully detected multiple SSLoad User-Agent events.

```text
04/18-14:43:26.036484  [**] [1:10000007:1] Potential SSLoad activity via User-Agent [**] [Priority: 0] {TCP} 10.4.18.169:49879 -> 85.239.53.219:80
04/18-14:53:05.959672  [**] [1:10000007:1] Potential SSLoad activity via User-Agent [**] [Priority: 0] {TCP} 10.4.18.169:49898 -> 85.239.53.219:80
04/18-15:03:07.802327  [**] [1:10000007:1] Potential SSLoad activity via User-Agent [**] [Priority: 0] {TCP} 10.4.18.169:49907 -> 85.239.53.219:80
04/18-15:13:00.760263  [**] [1:10000007:1] Potential SSLoad activity via User-Agent [**] [Priority: 0] {TCP} 10.4.18.169:49944 -> 85.239.53.219:80
04/18-15:23:03.777575  [**] [1:10000007:1] Potential SSLoad activity via User-Agent [**] [Priority: 0] {TCP} 10.4.18.169:49950 -> 85.239.53.219:80
04/18-15:33:03.910824  [**] [1:10000007:1] Potential SSLoad activity via User-Agent [**] [Priority: 0] {TCP} 10.4.18.169:49967 -> 85.239.53.219:80
04/18-15:43:10.838535  [**] [1:10000007:1] Potential SSLoad activity via User-Agent [**] [Priority: 0] {TCP} 10.4.18.169:49976 -> 85.239.53.219:80
04/18-15:53:05.821350  [**] [1:10000007:1] Potential SSLoad activity via User-Agent [**] [Priority: 0] {TCP} 10.4.18.169:49988 -> 85.239.53.219:80
04/18-16:03:00.817676  [**] [1:10000007:1] Potential SSLoad activity via User-Agent [**] [Priority: 0] {TCP} 10.4.18.169:50003 -> 85.239.53.219:80
```

---

## Alert Analysis

| Field               | Value                    |
| ------------------- | ------------------------ |
| Source IP           | `10.4.18.169`            |
| Destination IP      | `85.239.53.219`          |
| Destination Port    | `80`                     |
| Protocol            | TCP/HTTP                 |
| Detection Indicator | `User-Agent: SSLoad/1.1` |
| Rule SID            | `10000007`               |

The alerts showed repeated HTTP communication from the internal host:

```text
10.4.18.169
```

to the external server:

```text
85.239.53.219
```

over port `80`.

The repeated alerts appeared approximately every 10 minutes, which may indicate beaconing or periodic check-in behavior.

---

## Step 5: Verified Snort Log Creation

After the detection test, I checked the log directory.

```bash
cd ~/log
ls
```

Output:

```text
snort.log.1778827911
```

This confirmed that Snort generated a log file for the detected traffic.

---

## Security Relevance

SSLoad is associated with suspicious loader activity. Detecting unique User-Agent strings can help identify malware-related HTTP communication, especially when the User-Agent is unusual or linked to known malware behavior.

This rule provides a useful way to detect SSLoad-like activity in HTTP traffic using header-based inspection.

---

## Key Lessons Learned

* How to detect suspicious User-Agent values with Snort
* How to use `http_header` for HTTP header inspection
* How to use hexadecimal encoding for special characters
* How to perform offline PCAP detection with Snort
* How to identify repeated suspicious communication patterns
* How User-Agent strings can act as malware indicators

---

## Skills Practiced

* Custom Snort rule writing
* Malware traffic detection
* HTTP header inspection
* Offline PCAP analysis
* Alert interpretation
* User-Agent-based detection engineering
* SOC-style traffic analysis

---

## Conclusion

This lab demonstrated how Snort can detect potential SSLoad activity by inspecting HTTP User-Agent headers. The custom rule successfully triggered multiple alerts from the PCAP, showing repeated communication from an internal host to an external server using the suspicious `SSLoad/1.1` User-Agent.

```
