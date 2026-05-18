
## Snort Rule: Detecting SSH Brute Force Attacks

## Objective

The goal of this lab was to create a Snort rule capable of detecting a possible SSH brute force attack by identifying repeated SSH connection attempts within a short period of time.

The detection logic was based on observing multiple SSH connection attempts occurring rapidly in a PCAP file previously analyzed in Wireshark.

---

## Step 1: Cleared Old Snort Logs

I first cleared the existing Snort logs to ensure only the current lab activity would be logged.

```bash
cd ~/log
ls
sudo rm *
ls
````

---

## Step 2: Reviewed the PCAP in Wireshark

I opened the PCAP file in Wireshark to analyze the SSH traffic.

```bash
sudo wireshark -r 3.pcap
```

During the analysis, I observed multiple SSH connection attempts occurring repeatedly within approximately 25 seconds, which strongly indicated brute force behavior.

---

## Step 3: Created the SSH Brute Force Detection Rule

I edited the local Snort rules file:

```bash
sudo nano rules/local.rules
```

I created the following rule using Snorpy:

```snort
alert tcp any any -> any 22 (msg:"Possible SSH Brute Force Attack"; flow:to_server,established; threshold:type both, track by_src, count 5, seconds 30; sid:10000008; rev:1;)
```

---

## Rule Breakdown

| Component                    | Meaning                                              |
| ---------------------------- | ---------------------------------------------------- |
| `alert`                      | Generate an alert                                    |
| `tcp`                        | Inspect TCP traffic                                  |
| `any any -> any 22`          | Detect traffic targeting SSH port 22                 |
| `flow:to_server,established` | Match established client-to-server traffic           |
| `threshold`                  | Control alert frequency                              |
| `type both`                  | Trigger and limit alerts based on threshold behavior |
| `track by_src`               | Track activity by source IP                          |
| `count 5`                    | Trigger after 5 events                               |
| `seconds 30`                 | Within a 30-second window                            |
| `sid:10000008`               | Custom Snort rule ID                                 |
| `rev:1`                      | Rule revision                                        |

---

## Step 4: Ran Snort Against the PCAP

I executed Snort against the PCAP file:

```bash
sudo snort -c etc/snort.conf -r ~/Desktop/02_Network_Security/03_Snort/PCAPs/3.pcap -l ~/log -q -A console
```

Snort generated a single alert:

```text
06/07-10:22:40.428841  [**] [1:10000008:1] Possible SSH Brute Force Attack [**] [Priority: 0] {TCP} 192.168.1.7:54494 -> 192.168.1.6:22
```

---

## Understanding `threshold:type both`

The reason only one alert was generated was because the threshold type was set to:

```snort
type both
```

This mode both:

* triggers after the threshold is reached
* suppresses additional alerts within the configured time window

This reduces alert flooding and helps prevent excessive duplicate alerts.

---

## Step 5: Modified the Threshold Behavior

To observe all matching SSH attempts, I edited the rule again and changed:

```snort
type both
```

to:

```snort
type threshold
```

After saving the changes, I reran Snort against the same PCAP.

---

## Multiple Alerts Generated

This time Snort generated multiple alerts for the repeated SSH connection attempts.

Example alerts:

```text
06/07-10:22:40.428841  [**] [1:10000008:1] Possible SSH Brute Force Attack [**] [Priority: 0] {TCP} 192.168.1.7:54494 -> 192.168.1.6:22

06/07-10:22:40.433268  [**] [1:10000008:1] Possible SSH Brute Force Attack [**] [Priority: 0] {TCP} 192.168.1.7:54536 -> 192.168.1.6:22

06/07-10:22:48.933810  [**] [1:10000008:1] Possible SSH Brute Force Attack [**] [Priority: 0] {TCP} 192.168.1.7:54520 -> 192.168.1.6:22

06/07-10:23:03.357916  [**] [1:10000008:1] Possible SSH Brute Force Attack [**] [Priority: 0] {TCP} 192.168.1.7:37296 -> 192.168.1.6:22
```

---

## Alert Analysis

| Field            | Value                            |
| ---------------- | -------------------------------- |
| Source IP        | `192.168.1.7`                    |
| Destination IP   | `192.168.1.6`                    |
| Destination Port | `22`                             |
| Protocol         | TCP/SSH                          |
| Detection Type   | Repeated SSH connection attempts |
| Rule SID         | `10000008`                       |

The repeated SSH connection attempts from a single source host strongly indicated brute force behavior.

---

## Step 6: Verified Snort Log Files

After both Snort executions, I checked the log directory:

```bash
cd ~/log
ls
```

Output:

```text
snort.log.1778996215
snort.log.1778996409
```

Two separate log files were created because Snort was run twice against the PCAP.

---

## Security Relevance

SSH brute force attacks are common techniques used by attackers to gain unauthorized access to systems through repeated password guessing attempts.

This type of detection is important because:

* SSH is widely exposed on networks
* Brute force attacks are frequently automated
* Repeated login attempts can indicate credential attacks
* Thresholding helps identify suspicious behavior patterns

---

## Key Lessons Learned

* How to detect SSH brute force activity using Snort
* How to use `flow` keywords in rules
* How thresholding works in Snort
* Difference between `threshold:type both` and `threshold:type threshold`
* How to reduce or increase alert volume
* How to detect behavioral attack patterns instead of simple signatures

---

## Skills Practiced

* Snort thresholding
* Behavioral detection engineering
* SSH attack detection
* PCAP traffic analysis
* Wireshark traffic inspection
* Rate-based IDS detection
* SOC alert tuning
* Brute force attack analysis

---

## Conclusion

This lab demonstrated how Snort can detect SSH brute force attacks by identifying repeated SSH connection attempts within a short time period. By experimenting with different threshold types, I learned how Snort can either suppress excessive alerts or generate detailed event visibility depending on the configured detection strategy.

```


