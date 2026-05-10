# Snort Community Rules and Fast Alert Mode Lab

## Objective

The objective of this lab was to:

* Download and extract Snort community rules
* Explore the structure of community rule files
* Open Snort log files with Wireshark
* Manage and clean Snort log files
* Configure and run Snort in Fast Alert Mode
* Understand and troubleshoot Snort alert mode command syntax

---

# Step 1: Creating a Directory for Community Rules

A dedicated directory named `community` was created inside the Snort rules directory to store the downloaded community rules.

Command used:

```bash
mkdir community
```

The rules directory was verified using:

```bash
ls
```

Output:

```bash
black_list.rules  community  local.rules  white_list.rules
```

The newly created directory was then accessed:

```bash
cd community
```

---

# Step 2: Downloading Snort Community Rules

The official Snort community rules archive was downloaded from the Snort website using `wget`.

Command used:

```bash
sudo wget https://www.snort.org/downloads/community/community-rules.tar.gz
```

The download completed successfully and saved the file as:

```bash
community-rules.tar.gz
```

The downloaded file was confirmed using:

```bash
ls
```

Output:

```bash
community-rules.tar.gz
```

---

# Step 3: Extracting the Community Rules Archive

An initial extraction attempt failed because an invalid `tar` option (`y`) was used.

Incorrect commands attempted:

```bash
sudo tar xzyf community-rules.tar.gz
```

and

```bash
sudo tar -xzyf community-rules.tar.gz
```

Both commands generated the following error:

```bash
tar: invalid option -- 'y'
```

The issue was corrected by using the proper extraction syntax:

```bash
sudo tar -xzvf community-rules.tar.gz
```

The extraction completed successfully with the following output:

```bash
community-rules/
community-rules/community.rules
community-rules/VRT-License.txt
community-rules/LICENSE
community-rules/AUTHORS
community-rules/snort.conf
community-rules/sid-msg.map
```

The extraction result was verified using:

```bash
ls
```

Output:

```bash
community-rules  community-rules.tar.gz
```

---

# Step 4: Exploring the Extracted Community Rules

The extracted directory was entered:

```bash
cd community-rules
```

The contents were listed using:

```bash
ls
```

Output:

```bash
AUTHORS  community.rules  LICENSE  sid-msg.map  snort.conf  VRT-License.txt
```

The main community rules file was then inspected using:

```bash
cat community.rules | less
```

This allowed examination of the prebuilt Snort detection signatures provided by the Snort community.

---

# Step 5: Reviewing Existing Snort Log Files

The Snort log directory was accessed:

```bash
cd ~/log
```

Existing packet capture logs were listed:

```bash
ls
```

Output:

```bash
snort.log.1778137269
snort.log.1778218241
snort.log.1778314262
snort.log.1778314373
snort.log.1778317539
snort.log.1778386952
```

---

# Step 6: Opening Snort Packet Captures in Wireshark

One of the Snort-generated packet capture files was opened using Wireshark.

Command used:

```bash
sudo wireshark snort.log.1778386952
```

A warning appeared:

```bash
Warning: program compiled against libxml 215 using older 214
```

Despite the warning, Wireshark successfully opened the packet capture file for inspection.

This demonstrated that Snort log files are stored in pcap-compatible format and can be analyzed using Wireshark.

---

# Step 7: Cleaning the Log Directory

An attempt was made to remove all log files using:

```bash
rm *
```

The shell requested confirmation because the files were write-protected and owned by root.

The deletion process was interrupted midway using `CTRL + C`.

The remaining files were verified using:

```bash
ls
```

Detailed file permissions were then inspected:

```bash
ls -la
```

Output:

```bash
-rw-------  1 root root
```

This confirmed that the files were owned by the root user.

The issue was resolved by deleting the files with elevated privileges:

```bash
sudo rm *
```

The directory was then verified to be empty:

```bash
ls
```

---

# Step 8: Running Snort in Fast Alert Mode

The working directory was changed back to the Snort installation directory:

```bash
cd ~/snort-2.9.20
```

An incorrect attempt was made to start Snort using both `fast` and `console` alert modes simultaneously.

Command attempted:

```bash
sudo snort -A fast console -l ~/log -c etc/snort.conf -q
```

Another attempt included the interface:

```bash
sudo snort -A fast console -l ~/log -i eth0 -c etc/snort.conf -q
```

Both commands generated the following error:

```bash
ERROR: Can't set DAQ BPF filter to 'console'
```

---

# Understanding the Error

The error occurred because:

```bash
-A fast
```

already specifies the Snort alert mode.

Adding:

```bash
console
```

afterward caused Snort to interpret `console` as a Berkeley Packet Filter (BPF) expression rather than an alert mode.

---

# Correct Fast Alert Mode Command

The issue was corrected by using only the `fast` alert mode:

```bash
sudo snort -A fast -l ~/log -i eth0 -c etc/snort.conf -q
```

Snort started successfully in Fast Alert Mode.

The process was later stopped manually using:

```bash
CTRL + C
```

Output:

```bash
*** Caught Int-Signal
```

---

# Step 9: Verifying Generated Alert Files

The log directory was inspected after running Snort:

```bash
cd ~/log
```

Detailed contents were listed:

```bash
ls -la
```

Output:

```bash
-rw-r--r--  1 root root 760 alert
-rw-------  1 root root 374 snort.log.1778392005
```

This confirmed that:

* an `alert` file was generated,
* and a new packet capture log was created.

---

# Step 10: Reading the Fast Alert Output

The contents of the generated alert file were displayed using:

```bash
cat alert
```

Output:

```bash
05/10-01:47:20.600590  [**] [1:10000002:1] Connection to Remote IP on Port 4444 detected [**] [Priority: 0] {TCP} 10.0.2.15:3021 -> 172.66.147.243:4444
05/10-01:47:21.604063  [**] [1:10000002:1] Connection to Remote IP on Port 4444 detected [**] [Priority: 0] {TCP} 10.0.2.15:3022 -> 172.66.147.243:4444
05/10-01:47:22.605455  [**] [1:10000002:1] Connection to Remote IP on Port 4444 detected [**] [Priority: 0] {TCP} 10.0.2.15:3023 -> 172.66.147.243:4444
05/10-01:47:23.606566  [**] [1:10000002:1] Connection to Remote IP on Port 4444 detected [**] [Priority: 0] {TCP} 10.0.2.15:3024 -> 172.66.147.243:4444
05/10-01:47:24.607141  [**] [1:10000002:1] Connection to Remote IP on Port 4444 detected [**] [Priority: 0] {TCP} 10.0.2.15:3025 -> 172.66.147.243:4444
```

These alerts confirmed that the custom Snort rule detecting TCP traffic on port `4444` was functioning successfully in Fast Alert Mode.

---

# Key Concepts Learned

During this lab, the following concepts were learned:

* Downloading Snort community rules
* Extracting `.tar.gz` archives
* Understanding Snort community rule packages
* Viewing Snort rules
* Reading Snort packet capture logs using Wireshark
* Managing root-owned log files
* Understanding Fast Alert Mode
* Troubleshooting Snort command syntax errors
* Interpreting Snort-generated alert logs

---

# Tools Used

* Snort
* Wireshark
* Linux utilities:

  * `wget`
  * `tar`
  * `rm`
  * `cat`
  * `less`
  * `ls`

---

# Conclusion

This lab successfully demonstrated how to download and manage Snort community rules, analyze Snort-generated packet captures with Wireshark, clean and manage Snort log files, and properly configure Snort Fast Alert Mode. The lab also reinforced troubleshooting techniques related to Snort command-line syntax and alert configuration.
