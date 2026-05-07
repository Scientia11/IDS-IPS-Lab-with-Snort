## 03 - Running Snort in Packet Dump Mode

## Project Context

After successfully installing and configuring Snort 2.9.20, I tested Snort in packet dump mode. The purpose of this lab was to observe live network traffic, inspect packet headers, view Ethernet information, analyze ICMP traffic, and inspect packet payloads in hexadecimal and ASCII format.

Packet dump mode helped me understand how Snort sees network traffic before moving into rule-based IDS detection.

---

## Lab Environment

| Item | Details |
|---|---|
| Operating System | Kali Linux |
| IDS Tool | Snort |
| Version | Snort 2.9.20 GRE Build 82 |
| Network Interface | eth0 |
| Local IP Address | 10.0.2.15 |
| Network Range | 10.0.2.0/24 |
| Mode Tested | Packet Dump Mode |

---

## Step 1: Identified the Active Network Interface

Before capturing traffic, I checked my available network interfaces.

```bash
ifconfig
````

Relevant output:

```text
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 10.0.2.15  netmask 255.255.255.0  broadcast 10.0.2.255
        ether 08:00:27:70:af:7d
```

The active interface was identified as:

```text
eth0
```

The local IP address was:

```text
10.0.2.15
```

---

## Step 2: Ran Snort in Basic Packet Dump Mode

I started Snort in packet dump mode using:

```bash
sudo snort -i eth0
```

Snort initialized successfully and displayed:

```text
Running in packet dump mode
pcap DAQ configured to passive.
Acquiring network traffic from "eth0".
Decoding Ethernet
```

This confirmed that Snort was capturing live traffic from the `eth0` interface.

Example captured TCP packet:

```text
10.0.2.15:54224 -> 192.178.223.188:5228
TCP TTL:64 TOS:0x0 ID:37243 IpLen:20 DgmLen:40 DF
***A**** Seq: 0x7007DA0F  Ack: 0x1850BD1A  Win: 0xFFFF  TcpLen: 20
```

### Observation

Snort displayed packet metadata such as:

* source IP address
* destination IP address
* source and destination ports
* protocol
* TTL
* TCP flags
* sequence and acknowledgment numbers

---

## Step 3: Captured Ethernet Header Information

Next, I captured packets with Ethernet header information using the `-e` option.

```bash
sudo snort -i eth0 -e
```

Explanation:

| Option    | Meaning                               |
| --------- | ------------------------------------- |
| `-i eth0` | Capture traffic on the eth0 interface |
| `-e`      | Display Ethernet header information   |

---

## Step 4: Generated ICMP Traffic

In another terminal, I generated ICMP traffic by pinging Google DNS.

```bash
ping 8.8.8.8
```

Snort captured the ICMP echo request and reply.

Example ICMP echo request:

```text
08:00:27:70:AF:7D -> 52:55:0A:00:02:02 type:0x800 len:0x62
10.0.2.15 -> 8.8.8.8 ICMP TTL:64 TOS:0x0 ID:21488 IpLen:20 DgmLen:84 DF
Type:8  Code:0  ID:3   Seq:1  ECHO
```

Example ICMP echo reply:

```text
52:55:0A:00:02:02 -> 08:00:27:70:AF:7D type:0x800 len:0x62
8.8.8.8 -> 10.0.2.15 ICMP TTL:255 TOS:0x0 ID:31253 IpLen:20 DgmLen:84 DF
Type:0  Code:0  ID:3  Seq:1  ECHO REPLY
```

### ICMP Analysis

| Field    | Meaning              |
| -------- | -------------------- |
| `Type:8` | ICMP Echo Request    |
| `Type:0` | ICMP Echo Reply      |
| `Code:0` | Normal ICMP message  |
| `Seq`    | ICMP sequence number |

### Ethernet Analysis

```text
08:00:27:70:AF:7D -> 52:55:0A:00:02:02
```

This showed:

* source MAC address
* destination MAC address

```text
type:0x800
```

This indicated that the Ethernet frame was carrying IPv4 traffic.

---

## Step 5: Captured Packet Payloads in Hex and ASCII

Next, I captured packets with payload data using the `-d` option.

```bash
sudo snort -i eth0 -d
```

Explanation:

| Option    | Meaning                                 |
| --------- | --------------------------------------- |
| `-i eth0` | Capture traffic on eth0                 |
| `-d`      | Display packet payload in hex and ASCII |

---

## Step 6: Generated HTTP Traffic

In another terminal, I generated unencrypted HTTP traffic using `curl`.

```bash
curl http://example.com
```

This generated:

* DNS queries
* DNS responses
* TCP handshake packets
* HTTP GET request
* HTTP response

---

## Step 7: Observed DNS Query

Snort captured a DNS query from my Kali machine to Google DNS.

```text
10.0.2.15:42379 -> 8.8.8.8:53
UDP TTL:64 TOS:0x0 ID:37400 IpLen:20 DgmLen:57 DF
Len: 29
F1 3C 01 00 00 01 00 00 00 00 00 00 07 65 78 61  .<...........exa
6D 70 6C 65 03 63 6F 6D 00 00 01 00 01           mple.com.....
```

### Observation

The ASCII output revealed the queried domain:

```text
example.com
```

This showed that Snort can display readable payload content from DNS traffic.

---

## Step 8: Observed DNS Response

Snort also captured the DNS response from `8.8.8.8`.

```text
8.8.8.8:53 -> 10.0.2.15:42379
UDP TTL:64 TOS:0x0 ID:31748 IpLen:20 DgmLen:89
Len: 61
F1 3C 81 80 00 01 00 02 00 00 00 00 07 65 78 61  .<...........exa
6D 70 6C 65 03 63 6F 6D 00 00 01 00 01 C0 0C 00  mple.com........
```

This confirmed that Snort captured both DNS requests and DNS responses.

---

## Step 9: Observed TCP Three-Way Handshake

After DNS resolution, the system initiated a TCP connection to the web server on port 80.

### SYN Packet

```text
10.0.2.15:34412 -> 104.20.23.154:80
TCP TTL:64 TOS:0x0 ID:50191 IpLen:20 DgmLen:60 DF
******S* Seq: 0x68534BF8  Ack: 0x0  Win: 0xFAF0  TcpLen: 40
```

The flag `******S*` indicates a SYN packet.

### SYN-ACK Packet

```text
104.20.23.154:80 -> 10.0.2.15:34412
TCP TTL:64 TOS:0x8 ID:31750 IpLen:20 DgmLen:44
***A**S* Seq: 0x41FB9A01  Ack: 0x68534BF9  Win: 0xFFFF  TcpLen: 24
```

The flags `***A**S*` indicate SYN and ACK.

### ACK Packet

```text
10.0.2.15:34412 -> 104.20.23.154:80
TCP TTL:64 TOS:0x0 ID:50193 IpLen:20 DgmLen:40 DF
***A**** Seq: 0x68534BF9  Ack: 0x41FB9A02  Win: 0xFAF0  TcpLen: 20
```

The flag `***A****` indicates ACK.

### Observation

These three packets represent the TCP three-way handshake:

```text
SYN -> SYN-ACK -> ACK
```

---

## Step 10: Observed HTTP GET Request

After the TCP handshake completed, Snort captured the HTTP GET request created by `curl`.

```text
10.0.2.15:34412 -> 104.20.23.154:80
TCP TTL:64 TOS:0x0 ID:50194 IpLen:20 DgmLen:115 DF
***AP*** Seq: 0x68534BF9  Ack: 0x41FB9A02  Win: 0xFAF0  TcpLen: 20
47 45 54 20 2F 20 48 54 54 50 2F 31 2E 31 0D 0A  GET / HTTP/1.1..
48 6F 73 74 3A 20 65 78 61 6D 70 6C 65 2E 63 6F  Host: example.co
6D 0D 0A 55 73 65 72 2D 41 67 65 6E 74 3A 20 63  m..User-Agent: c
75 72 6C 2F 38 2E 31 37 2E 30 0D 0A 41 63 63 65  url/8.17.0..Acce
70 74 3A 20 2A 2F 2A 0D 0A 0D 0A                 pt: */*....
```

Readable ASCII output included:

```text
GET / HTTP/1.1
Host: example.com
User-Agent: curl/8.17.0
Accept: */*
```

### Observation

Because the request used plain HTTP, Snort was able to display the request contents clearly.

---

## Step 11: Observed HTTP Response

Snort captured the HTTP response from the server.

```text
104.20.23.154:80 -> 10.0.2.15:34412
TCP TTL:64 TOS:0x8 ID:31752 IpLen:20 DgmLen:877
***AP*** Seq: 0x41FB9A02  Ack: 0x68534C44  Win: 0xFFFF  TcpLen: 20
48 54 54 50 2F 31 2E 31 20 32 30 30 20 4F 4B 0D  HTTP/1.1 200 OK.
```

Readable ASCII output included:

```text
HTTP/1.1 200 OK
Content-Type: text/html
Server: cloudflare
```

The response body also contained visible HTML from the Example Domain page.

---

## Important Warning Observed

During packet dump mode, Snort displayed:

```text
WARNING: No preprocessors configured for policy 0.
```

This warning appeared because Snort was running in packet dump/sniffer mode without loading the full IDS configuration and preprocessors.

This did not prevent Snort from capturing and displaying packets.

---

## Key Lessons Learned

This packet dump lab helped me understand:

* how to identify the correct network interface
* how to run Snort in basic packet dump mode
* how to display Ethernet frame information
* how to analyze ICMP echo requests and replies
* how to inspect packet payloads in hex and ASCII
* how DNS queries appear in packet payloads
* how TCP three-way handshakes appear in Snort output
* how unencrypted HTTP traffic can be inspected
* why HTTPS traffic is not readable in the same way as HTTP traffic

---

## Commands Used

### Check Network Interfaces

```bash
ifconfig
```

### Run Basic Packet Dump Mode

```bash
sudo snort -i eth0
```

### Run Packet Dump Mode with Ethernet Headers

```bash
sudo snort -i eth0 -e
```

### Generate ICMP Traffic

```bash
ping 8.8.8.8
```

### Run Packet Dump Mode with Payload Display

```bash
sudo snort -i eth0 -d
```

### Generate HTTP Traffic

```bash
curl http://example.com
```

---

## Outcome

Snort successfully captured and displayed live traffic in packet dump mode.

The lab confirmed that Snort can be used to observe:

* TCP packets
* UDP packets
* ICMP packets
* Ethernet headers
* DNS payloads
* HTTP requests
* HTTP responses
* TCP handshake behavior

This completed the packet dump mode phase of my Snort IDS/IPS lab.

