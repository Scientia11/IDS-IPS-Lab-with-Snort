## Snort IDS Lab – Detecting Executable Downloads Over HTTP

Today’s lab focused on improving Snort detection capabilities by identifying executable file downloads over HTTP traffic, even when the `.exe` extension is not explicitly visible in the URI.

## Objective

The goal of this lab was to detect executable file downloads by analyzing the HTTP response headers instead of relying only on file extensions in the URL.

In a previous lab, detection depended on `.exe` appearing directly in the URI. However, attackers may disguise downloads or hide file extensions. To improve detection accuracy, I used the HTTP `Content-Type` header to identify Windows executable downloads.


# Step 1 – Clearing Previous Snort Logs

Before starting the analysis, I cleared the existing Snort log directory.

```bash
cd ~/log
sudo rm *
```

This ensured that only alerts generated during the current session would appear in the logs.

---

# Step 2 – Reviewing the PCAP File in Wireshark

I opened the PCAP file in Wireshark to inspect the traffic and identify indicators associated with executable downloads.

```bash
sudo wireshark -r ~/Desktop/02_Network_Security/03_Snort/PCAPs/1.pcap
```

During analysis, I discovered that the downloaded file was:

```text
audiodg.exe
```

I also observed that the HTTP response used the content type:

```text
Content-Type: application/x-msdownload
```

This became the basis for the new Snort detection rule.

---

# Step 3 – Writing a New Snort Rule

I edited the `local.rules` file and created a rule to detect HTTP responses containing the executable content type.

Initial rule:

```snort
alert tcp any 80 -> any any (msg:"Potential .exe file download detected over HTTP"; content:"Content-Type: application/x-msdownload"; http_header; sid:10000005; rev:1;)
```

While testing the rule, Snort returned an error because the content value was not properly formatted.

Error received:

```text
Content data needs to be enclosed in quotation marks (")!
```

After correcting the syntax, the rule worked successfully.

---

# Step 4 – Improving the Rule

I later improved the rule for more reliable matching by including the colon (`:`) as hexadecimal and enabling case-insensitive matching.

Improved rule:

```snort
alert tcp any 80 -> any any (msg:"Potential Windows executable download over HTTP"; content:"Content-Type|3A| application/x-msdownload"; nocase; http_header; sid:10000005; rev:2;)
```

## Explanation of the Rule

| Component           | Purpose                                       |   |                                   |
| ------------------- | --------------------------------------------- | - | --------------------------------- |
| `alert`             | Generates an alert when matched               |   |                                   |
| `tcp`               | Inspects TCP traffic                          |   |                                   |
| `any 80 -> any any` | Detects traffic originating from HTTP port 80 |   |                                   |
| `msg`               | Displays the alert message                    |   |                                   |
| `content`           | Searches for the HTTP content type            |   |                                   |
| `                   | 3A                                            | ` | Hexadecimal representation of `:` |
| `nocase`            | Makes matching case-insensitive               |   |                                   |
| `http_header`       | Restricts inspection to HTTP headers          |   |                                   |
| `sid`               | Unique Snort rule ID                          |   |                                   |
| `rev`               | Rule revision number                          |   |                                   |

---

# Step 5 – Running Snort Against the PCAP

I tested the rule against the PCAP file using Snort in offline analysis mode.

```bash
sudo snort -c etc/snort.conf -r ~/Desktop/02_Network_Security/03_Snort/PCAPs/1.pcap -l ~/log -q -A console
```

Snort successfully generated an alert:

```text
[1:10000005:1] Potential .exe file download detected over HTTP
```

Alert details:

```text
09/14-10:35:32.825541  [**] [1:10000005:1] Potential .exe file download detected over HTTP [**] [Priority: 0] {TCP} 103.232.55.148:80 -> 10.0.0.168:49724
```

This confirmed that the rule correctly detected the executable download based on the HTTP response header.

---

# Step 6 – Reading the Snort Log File

After detection, I examined the logged packets in detail.

First attempt:

```bash
snort -r snort.log.1778679581 -q -d
```

This produced a permission error because the log file belonged to root.

Error:

```text
Permission denied
```

I corrected it by using `sudo`:

```bash
sudo snort -r snort.log.1778679581 -q -d
```

---

# Step 7 – Verifying the Executable Download

Inside the packet payload, I observed several indicators confirming that the downloaded content was a Windows executable.

## HTTP Header Evidence

```text
Content-Type: application/x-msdownload
```

## Windows Executable Indicators

```text
MZ
PE
This program cannot be run in DOS mode
```

These are well-known signatures found in Portable Executable (PE) files used by Windows applications.

---

# Key Concepts Learned

* Detecting malware downloads using HTTP headers
* Understanding HTTP response analysis
* Using `http_header` in Snort rules
* Matching content using hexadecimal notation
* Using `nocase` for flexible detection
* Offline PCAP analysis with Snort
* Reading and interpreting raw packet payloads
* Identifying Windows executable signatures (`MZ` and `PE`)


# Conclusion

In this lab, I improved my Snort detection skills by creating a rule capable of identifying executable downloads based on HTTP response headers rather than relying solely on file extensions in URLs.

The rule successfully detected the download of `audiodg.exe` from a PCAP file and demonstrated how Snort can be used for malware detection and network forensic analysis.
