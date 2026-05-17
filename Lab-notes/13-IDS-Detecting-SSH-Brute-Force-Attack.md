## Detecting SSH Brute Force Attacks Using Snort IDS

---

## 📖 Objective

The objective of this lab was to design and implement a Snort IDS rule capable of detecting SSH brute force attacks by analyzing suspicious repeated authentication attempts within a short time window.

---

## 🧪 Lab Methodology

### 1. Log Preparation

To ensure a clean test environment, existing Snort logs were cleared before analysis:

```bash
cd ~/log
sudo rm *
ls
```

This ensured only new alerts generated during testing would be captured.

---

### 2. Traffic Analysis (Wireshark)

The provided PCAP file (`3.pcap`) was opened in Wireshark to inspect SSH traffic behavior.

Key observation:

* Multiple SSH connection attempts were observed within ~25 seconds
* Repeated TCP connections targeting port **22**
* Behavior strongly indicated a **brute force attack pattern**

---

### 3. Snort Rule Development

A detection rule was created using Snorpy and implemented in `local.rules`:

```snort
alert tcp any any -> any 22 (
    msg:"Possible SSH Brute Force Attack";
    flow:to_server,established;
    threshold:type both, track by_src, count 5, seconds 30;
    sid:10000008;
    rev:1;
)
```

### 🔍 Rule Explanation

* **alert tcp** → Monitors TCP traffic
* **port 22** → Targets SSH service
* **flow:to_server,established** → Ensures valid SSH session attempts
* **threshold:type both** → Controls alert frequency (reduces noise)
* **track by_src** → Tracks attacks per source IP
* **count 5, seconds 30** → Triggers if 5+ attempts occur within 30 seconds
* **sid:10000008** → Unique Snort rule ID
* **rev:1** → Rule version

---

### 4. Snort Execution (PCAP Testing)

```bash
sudo snort -c etc/snort.conf \
-r ~/Desktop/02_Network_Security/03_Snort/PCAPs/3.pcap \
-l ~/log -q -A console
```

---

## 📊 Results & Observations

### 🔹 First Run (threshold: both)

* Only **1 alert generated**
* This behavior is expected because `threshold:type both` suppresses repeated alerts after the first trigger.

Example output:

```
Possible SSH Brute Force Attack
192.168.1.7 -> 192.168.1.6:22
```

---

### 🔹 Second Run (adjusted threshold behavior)

After modifying the threshold behavior, Snort generated **multiple alerts**, correctly reflecting repeated SSH connection attempts.

Example output:

* Multiple alerts within seconds
* Consistent source IP: `192.168.1.7`
* Target IP: `192.168.1.6:22`

---

## 📁 Log Evidence

After execution, Snort generated log files:

```bash
cd ~/log
ls
```

Output:

```
snort.log.1778996215
snort.log.1778996409
```

This confirms two separate Snort runs against the PCAP file.

---

## 🧠 Key Learning Outcome

This exercise demonstrates how Snort can be used to detect SSH brute force attacks by:

* Identifying repeated authentication attempts
* Applying time-window-based thresholding
* Reducing false positives using alert control mechanisms
* Validating detection rules using PCAP replay

---

## 🛡️ SOC Relevance

SSH brute force detection is critical in SOC environments because it helps identify:

* Credential stuffing attempts
* Automated attack tools (Hydra, Ncrack, etc.)
* Early-stage intrusion attempts

---

## 📌 Conclusion

The implemented Snort rule successfully detected SSH brute force behavior in captured network traffic. Adjusting threshold parameters significantly affected alert output behavior, highlighting the importance of fine-tuning IDS rules for operational accuracy.
