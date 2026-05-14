## Snort Rule: Detecting Executable Files Using File Signature

## Objective

The goal of this lab was to detect a Windows executable file download using its file signature instead of relying on the `.exe` extension or HTTP Content-Type header.

Windows executable files commonly begin with the DOS header signature:

```text
MZ
````

In hexadecimal:

```text
4D 5A
```

---

## Step 1: Cleared Old Snort Logs

```bash
cd ~/log
sudo rm *
ls
```

---

## Step 2: Created File Signature Detection Rule

I researched the Windows executable file signature and created this Snort rule:

```snort
alert tcp any 80 -> any any (msg:"HTTP payload contains DOS MZ or PE executable file signature"; file_data; content:"|4D 5A|"; depth:2; sid:10000006; rev:1;)
```

---

## Rule Breakdown

| Component           | Meaning                        |    |                                     |
| ------------------- | ------------------------------ | -- | ----------------------------------- |
| `alert`             | Generate an alert              |    |                                     |
| `tcp`               | Inspect TCP traffic            |    |                                     |
| `any 80 -> any any` | HTTP server response to client |    |                                     |
| `file_data`         | Inspect normalized file data   |    |                                     |
| `content:"          | 4D 5A                          | "` | Match the `MZ` executable signature |
| `depth:2`           | Look only at the first 2 bytes |    |                                     |
| `sid:10000006`      | Custom rule ID                 |    |                                     |
| `rev:1`             | Rule revision                  |    |                                     |

---

## Step 3: First Rule Error

When I first ran Snort, I received this error:

```text
Rule option depth uses an undefined byte_extract variable name
```

This was caused by incorrect rule formatting around the `depth` option.

After correcting the rule syntax, Snort ran successfully.

---

## Step 4: Ran Snort Against the PCAP

```bash
sudo snort -c etc/snort.conf -r ~/Desktop/02_Network_Security/03_Snort/PCAPs/1.pcap -l ~/log -q -A console
```

Snort generated the alert:

```text
09/14-10:35:32.825541  [**] [1:10000006:1] HTTP payload contains DOS MZ or PE executable file signature [**] [Priority: 0] {TCP} 103.232.55.148:80 -> 10.0.0.168:49724
```

---

## Step 5: Opened Snort Log in Wireshark

```bash
cd ~/log
ls
sudo wireshark -r snort.log.1778788314
```

In Wireshark, I followed the TCP stream and confirmed that the traffic contained a Windows executable file.

---

## Key Lesson

This rule is stronger than checking only for `.exe` in the URI because it detects the actual executable file signature inside the payload.

This helps detect executable downloads even when the filename or extension is hidden, changed, or not shown clearly in the URL.

---

## Skills Practiced

* File signature detection
* Snort `file_data` inspection
* Hexadecimal content matching
* Offline PCAP analysis
* Wireshark TCP stream analysis
* Malware download detection logic


