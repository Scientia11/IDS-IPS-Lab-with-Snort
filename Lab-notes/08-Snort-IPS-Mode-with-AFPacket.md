# Snort IPS Mode with AFPacket

## Objective

The objective of this lab was to transition from Snort IDS mode into full IPS (Intrusion Prevention System) mode using the AFPacket DAQ module.
The goal was to configure Snort to actively block FTP traffic instead of only detecting it.

---

# Writing an IPS Rule to Drop FTP Traffic

I first opened the `local.rules` file and commented out all previously created IDS rules.
I then created a new IPS rule to block FTP traffic on port 21.

## IPS Rule

```snort
#LOCAL RULES

#IDS MODE
#alert icmp any any -> 8.8.8.8 any (msg: "ICMP traffic to 8.8.8.8 detected"; sid: 10000001; rev: 1;)

#alert tcp any any -> any 4444 (msg: "Connection to Remote IP on Port 4444 detected"; sid: 10000002; rev: 1;)

#IPS MODE
drop tcp any any <> any 21 (msg:"Drop FTP traffic"; sid:10000003; rev:1;)
```

---

# Rule Explanation

| Component | Purpose                             |
| --------- | ----------------------------------- |
| `drop`    | Block matching traffic              |
| `tcp`     | Applies rule to TCP traffic         |
| `any any` | Match any source IP and source port |
| `<>`      | Bidirectional traffic matching      |
| `any 21`  | Match traffic involving FTP port 21 |
| `msg`     | Alert message                       |
| `sid`     | Unique Snort rule ID                |
| `rev`     | Rule revision number                |

---

# Exploring Snorpy Rule Generator

During the lab, I also explored:

[Snorpy Rule Generator](https://snorpy.cyb3rs3c.net?utm_source=chatgpt.com)

This tool helps generate Snort rules automatically through a graphical interface.

I used the platform to recreate the same FTP drop rule and compare it with the manually written version.

---

# Attempting to Run Snort in IPS Mode

I initially attempted to run Snort in IPS mode using:

```bash
sudo snort -q -A console -i eth0 -c etc/snort.conf -Q
```

Snort returned the following error:

```text
ERROR: pcap DAQ does not support inline.
Fatal Error, Quitting..
```

---

# Understanding the Error

The error occurred because the default `pcap` DAQ only supports passive sniffing (IDS mode).

Inline IPS mode requires an inline-capable DAQ such as:

* AFPacket
* NFQ

---

# Resolving the DAQ Inline Error

After researching the issue, I discovered that AFPacket inline mode requires:

* At least two network interfaces
* Interface bridging for packet forwarding

---

# Configuring Multiple Network Interfaces in VirtualBox

To resolve the issue, I configured a second network adapter in VirtualBox.

## Steps Performed

### 1. Powered Off the Kali Linux VM

The VM was completely shut down before modifying network settings.

---

### 2. Opened VirtualBox Settings

Opened the settings menu for the Kali Linux VM.

---

### 3. Navigated to the Network Tab

Selected:

```text
Settings → Network
```

---

### 4. Enabled Adapter 2

Under Adapter 2:

* Enabled:

```text
Enable Network Adapter
```

---

### 5. Configured Adapter Name

Changed the adapter name to:

```text
ADDO
```

This matched the configuration used by Adapter 1.

---

### 6. Restarted the VM

Clicked:

```text
OK
```

and restarted Kali Linux.

---

# Verifying the New Interface

After rebooting the VM, I verified the interfaces using:

```bash
ifconfig
```

---

# Result

A new interface named `eth1` appeared alongside the original `eth0`.

## Interface Summary

### eth0

* Original interface
* Previously used during IDS labs

### eth1

* Newly added interface
* Received IP address:

```text
10.0.3.15
```

---

# Why Two Interfaces Are Important

AFPacket IPS mode requires two interfaces so Snort can:

* Receive traffic on one interface
* Forward traffic through another interface
* Inspect packets inline
* Drop malicious packets in real time

---

# Running Snort in IPS Mode with AFPacket

After configuring both interfaces, I started Snort in inline IPS mode using AFPacket.

## Command Used

```bash
sudo snort -q -A console -i eth0:eth1 -c etc/snort.conf --daq afpacket -Q
```

---

# Command Breakdown

| Option              | Purpose                           |
| ------------------- | --------------------------------- |
| `-q`                | Quiet mode                        |
| `-A console`        | Display alerts in terminal        |
| `-i eth0:eth1`      | Bridge traffic between interfaces |
| `-c etc/snort.conf` | Specify Snort configuration file  |
| `--daq afpacket`    | Use AFPacket DAQ module           |
| `-Q`                | Enable inline IPS mode            |

---

# Testing the FTP Drop Rule

To test the rule, I attempted to connect to the Rebex FTP test server using:

```bash
ftp test.rebex.net
```

---

# Snort Successfully Blocked FTP Traffic

When the FTP connection was attempted while Snort was running in IPS mode, Snort immediately blocked the traffic and generated drop alerts.

## Alert Output

```text
05/11-01:31:36.406130  [Drop] [**] [1:10000003:1] Drop FTP traffic [**] [Priority: 0] {TCP} 194.108.117.16:21 -> 10.0.3.15:46158
```

```text
05/11-01:32:36.432586  [Drop] [**] [1:10000003:1] Drop FTP traffic [**] [Priority: 0] {TCP} 194.108.117.16:21 -> 10.0.3.15:46158
```

---

# Verifying IPS Functionality

## Test 1 — Snort Running

* Started Snort in IPS mode
* Attempted FTP connection
* Snort blocked the traffic

---

## Test 2 — Snort Stopped

* Stopped Snort using:

```text
Ctrl + C
```

* Attempted FTP connection again
* FTP connection worked successfully

---

## Test 3 — Snort Restarted

* Restarted Snort in IPS mode
* Attempted FTP connection again
* Snort blocked the traffic again

## Additional Alerts

```text
05/11-01:36:04.300339  [Drop] [**] [1:10000003:1] Drop FTP traffic [**] [Priority: 0] {TCP} 194.108.117.16:21 -> 10.0.3.15:47492
```

```text
05/11-01:37:04.331904  [Drop] [**] [1:10000003:1] Drop FTP traffic [**] [Priority: 0] {TCP} 194.108.117.16:21 -> 10.0.3.15:47492
```

This confirmed that Snort was actively functioning as an Intrusion Prevention System (IPS).

---

# Key Learning Points

* IDS mode only detects attacks
* IPS mode can actively block traffic
* `pcap` DAQ does not support inline mode
* AFPacket enables inline packet inspection
* Two network interfaces are required for AFPacket bridging
* Snort can successfully block FTP traffic using `drop` rules
* IPS functionality can be verified by testing with and without Snort running

---

# Commands Used During the Lab

## Open Rules File

```bash
sudo nano local.rules
```

## Verify Interfaces

```bash
ifconfig
```

## Start Snort in IPS Mode

```bash
sudo snort -q -A console -i eth0:eth1 -c etc/snort.conf --daq afpacket -Q
```

## Test FTP Traffic

```bash
ftp test.rebex.net
```

---

# Skills Practiced

* Writing Snort IPS rules
* AFPacket DAQ configuration
* VirtualBox network adapter configuration
* Linux network interface management
* Inline packet inspection
* Traffic blocking with Snort
* FTP traffic analysis
* IPS troubleshooting
* Inline intrusion prevention deployment
