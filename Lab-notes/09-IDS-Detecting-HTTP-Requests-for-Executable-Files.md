
## Snort Rule: Detecting HTTP Requests for Executable Files

## Objective

The objective of this lab was to write and test a Snort rule that detects HTTP requests containing `.exe` in the URI. Executable file downloads are important to monitor because attackers commonly use `.exe` files to deliver malware, backdoors, trojans, and other malicious payloads.

---

## Rule Created

I created the rule using Snorpy and added it to my Snort rules file.

```snort
alert tcp any any -> any 80 (msg:"HTTP URL contains .exe"; content:"|2e|exe"; nocase; http_uri; sid:10000004; rev:1;)
````

---

## Rule Breakdown

| Rule Component | Meaning                                         |       |                                                          |
| -------------- | ----------------------------------------------- | ----- | -------------------------------------------------------- |
| `alert`        | Generate an alert when traffic matches          |       |                                                          |
| `tcp`          | Inspect TCP traffic                             |       |                                                          |
| `any any`      | Match any source IP and source port             |       |                                                          |
| `->`           | One-way traffic direction                       |       |                                                          |
| `any 80`       | Match traffic going to any host on HTTP port 80 |       |                                                          |
| `msg`          | Alert message displayed when triggered          |       |                                                          |
| `content:"     | 2e                                              | exe"` | Match `.exe` where `2e` is the hexadecimal value for `.` |
| `nocase`       | Match regardless of uppercase or lowercase      |       |                                                          |
| `http_uri`     | Search only inside the HTTP URI                 |       |                                                          |
| `sid:10000004` | Custom Snort rule ID                            |       |                                                          |
| `rev:1`        | Rule revision number                            |       |                                                          |

---

## Why This Rule Matters

This rule helps detect HTTP requests for executable files. This is useful because malicious files are often delivered through executable downloads over HTTP.

Examples that could trigger this rule include:

```text
/download/file.exe
/setup.EXE
/service/.audiodg.exe
```

---

## Testing Method

I tested the rule by running Snort against a PCAP file from a previous network traffic analysis session.

During that earlier analysis, I discovered traffic involving the download of an executable file named:

```text
audiodg.exe
```

---

## Command Used to Analyze the PCAP

```bash
sudo snort -q -c etc/snort.conf -r ~/Desktop/02_Network_Security/03_Snort/PCAPs/1.pcap -A console -l ~/log
```

### Command Breakdown

| Option              | Meaning                          |
| ------------------- | -------------------------------- |
| `-q`                | Quiet mode                       |
| `-c etc/snort.conf` | Use the Snort configuration file |
| `-r`                | Read packets from a PCAP file    |
| `-A console`        | Display alerts in the terminal   |
| `-l ~/log`          | Save logs to the log directory   |

---

## Alerts Generated

Snort generated two alerts:

```text
09/14-10:35:31.840413  [**] [1:10000004:1] HTTP URL contains .exe [**] [Priority: 0] {TCP} 10.0.0.168:49722 -> 13.107.5.80:80
09/14-10:35:32.552578  [**] [1:10000004:1] HTTP URL contains .exe [**] [Priority: 0] {TCP} 10.0.0.168:49724 -> 103.232.55.148:80
```

---

## Alert Analysis

| Alert   | Source             | Destination         | Reason                    |
| ------- | ------------------ | ------------------- | ------------------------- |
| Alert 1 | `10.0.0.168:49722` | `13.107.5.80:80`    | HTTP URI contained `.exe` |
| Alert 2 | `10.0.0.168:49724` | `103.232.55.148:80` | HTTP URI contained `.exe` |

Both alerts came from the internal host:

```text
10.0.0.168
```

This indicated that the host made HTTP requests involving an executable file.

---

## Inspecting the Snort Log File

After Snort generated the alerts, I opened the Snort log file to inspect the packet contents.

```bash
sudo snort -q -r ~/log/snort.log.1778565343 -d
```

### Command Breakdown

| Option | Meaning                            |
| ------ | ---------------------------------- |
| `-r`   | Read packets from a Snort log file |
| `-d`   | Display packet payload data        |

---

## First Packet Analysis

The first alert showed a request to:

```text
13.107.5.80:80
```

The packet payload contained:

```text
GET /qsml.aspx?query=http%3A%2F%2F103.232.55.148%2Fservice%2F.audiodg.exe
```

Important decoded content:

```text
http://103.232.55.148/service/.audiodg.exe
```

This showed that the HTTP URI contained a reference to the executable file:

```text
.audiodg.exe
```

---

## Second Packet Analysis

The second alert showed a direct HTTP request to:

```text
103.232.55.148:80
```

The packet payload contained:

```text
GET /service/.audiodg.exe HTTP/1.1
```

The host header showed:

```text
Host: 103.232.55.148
```

This confirmed a direct request to download the executable file:

```text
.audiodg.exe
```

---

## Key Observations

* Snort detected two HTTP requests containing `.exe`.
* The first alert appeared to involve a URL query containing the executable download path.
* The second alert showed the direct download request for `.audiodg.exe`.
* The rule successfully detected `.exe` in the HTTP URI.
* Because the traffic was HTTP, Snort could inspect the URI clearly.
* This detection would not work the same way on encrypted HTTPS traffic without TLS inspection.

---

## Security Relevance

Executable downloads are important to monitor because they may indicate:

* malware delivery,
* suspicious software installation,
* trojan download activity,
* command-and-control staging,
* or user-initiated download of potentially unsafe files.

The filename `.audiodg.exe` is suspicious because it resembles a Windows system process name. Attackers sometimes use names similar to legitimate Windows processes to avoid suspicion.

---

## Skills Practiced

* Writing Snort content-matching rules
* Using `http_uri`
* Detecting executable file downloads
* Reading PCAP files with Snort
* Analyzing Snort alerts
* Inspecting packet payloads
* Identifying suspicious HTTP requests
* Connecting IDS alerts to packet evidence


## Conclusion

This lab demonstrated how Snort can detect HTTP requests containing executable file names. The custom rule successfully triggered on traffic involving `.audiodg.exe`, confirming that Snort can identify potentially suspicious executable download activity in unencrypted HTTP traffic.

```
```
