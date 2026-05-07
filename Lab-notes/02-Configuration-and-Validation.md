## 02 - Snort Configuration and Validation

## Project Context

After successfully installing Snort 2.9.20 from source on Kali Linux, the next phase involved configuring Snort and validating the configuration file (`snort.conf`).

This stage included:
- configuring network variables,
- creating missing directories,
- creating missing rule files,
- resolving configuration errors,
- and validating the Snort setup.

The objective was to prepare Snort for rule creation and IDS testing.

---

# Lab Environment

| Item | Details |
|---|---|
| Operating System | Kali Linux |
| IDS Tool | Snort |
| Version | Snort 2.9.20 GRE Build 82 |
| Configuration File | `snort.conf` |
| Working Directory | `~/snort-2.9.20/etc` |

---

##edits interface settings & disable efault rules

# Step 1: Attempted Initial Configuration Validation

I tested the Snort configuration using:

```bash
sudo snort -T -c snort.conf
```

The `-T` option runs Snort in test mode to validate the configuration without starting packet inspection.

---

# Step 2: HOME_NET Variable Error

The first validation failed with this error:

```text
ERROR: snort.conf(46) Variable name should contain minimum 1 alphabetic character.
Following variable name is not allowed: 10.0.2.0/24
```

This happened because the `HOME_NET` variable was incorrectly configured.

---

# Step 3: Inspected the Configuration File

To inspect the problematic lines around line 46, I used:

```bash
nl -ba snort.conf | sed -n '40,55p'
```

The output showed:

```text
46  ipvar 10.0.2.0/24 any
```

The variable name `HOME_NET` was missing.

---

# Step 4: Corrected the HOME_NET Variable

I fixed the line using:

```bash
sed -i '46s/.*/ipvar HOME_NET 10.0.2.0\/24/' snort.conf
```

The corrected line became:

```bash
ipvar HOME_NET 10.0.2.0/24
```

This defined the protected network range for Snort.

---

# Step 5: Dynamic Rules Directory Error

After re-testing the configuration, I received this error:

```text
Could not stat dynamic module path "/usr/local/lib/snort_dynamicrules"
```

This happened because the required directory did not exist.

---

# Step 6: Created Missing Dynamic Rules Directory

I created the missing directory using:

```bash
sudo mkdir -p /usr/local/lib/snort_dynamicrules
```

---

# Step 7: Missing local.rules File Error

After another validation attempt, Snort reported:

```text
Unable to open rules file "./../rules/local.rules"
```

This occurred because the `rules` directory and `local.rules` file had not yet been created.

---

# Step 8: Created Rules Directory and local.rules File

I created both using:

```bash
mkdir -p ../rules && touch ../rules/local.rules
```

This created:

* the `rules` directory,
* and an empty `local.rules` file for custom IDS rules.

---

# Step 9: Missing Reputation Rule Files Error

The next validation produced this error:

```text
Unable to open address file ./../rules/white_list.rules
```

Snort also expected:

* `white_list.rules`
* `black_list.rules`

---

# Step 10: Created Reputation Rule Files

I created the missing files using:

```bash
touch ../rules/white_list.rules ../rules/black_list.rules
```

These files are used by the Snort reputation preprocessor.

---

# Step 11: Successfully Validated Snort Configuration

After resolving all missing files and directories, I ran:

```bash
sudo snort -T -c snort.conf
```

The configuration completed successfully.

Final validation output:

```text
Snort successfully validated the configuration!
```

---

# Important Warnings Observed

During validation, several warnings appeared:

## 1. Dynamic Libraries Warning

```text
WARNING: No dynamic libraries found in directory /usr/local/lib/snort_dynamicrules
```

This warning was acceptable because no additional dynamic detection rules had been installed yet.

---

## 2. Normalization Warnings

```text
WARNING: ip4 normalizations disabled because not inline.
WARNING: tcp normalizations disabled because not inline.
```

These warnings appeared because Snort was operating in IDS mode rather than inline IPS mode.

---

## 3. Reputation Warning

```text
WARNING: Can't find any whitelist/blacklist entries.
Reputation Preprocessor disabled.
```

This was expected because the whitelist and blacklist files were empty.

---

# Final Configuration Status

Snort configuration validation completed successfully.

## Verified Components

* Snort configuration file loaded successfully
* Dynamic preprocessors loaded successfully
* Rule directories configured successfully
* Rule files accessible
* HOME_NET configured correctly
* Logging directory functional

---

# Key Lessons Learned

This configuration phase helped me understand:

* the importance of Snort variables,
* how Snort loads rules and preprocessors,
* how to troubleshoot configuration validation errors,
* how Snort depends on required directories and rule files,
* and how to use Snort test mode for validation.

I also learned how Snort preprocessors initialize and how to interpret validation warnings versus fatal errors.

---

# Commands Used During Configuration

## Validate Configuration

```bash
sudo snort -T -c snort.conf
```

## Inspect Configuration Lines

```bash
nl -ba snort.conf | sed -n '40,55p'
```

## Fix HOME_NET

```bash
sed -i '46s/.*/ipvar HOME_NET 10.0.2.0\/24/' snort.conf
```

## Create Dynamic Rules Directory

```bash
sudo mkdir -p /usr/local/lib/snort_dynamicrules
```

## Create Rules Directory and local.rules

```bash
mkdir -p ../rules && touch ../rules/local.rules
```

## Create Reputation Rule Files

```bash
touch ../rules/white_list.rules ../rules/black_list.rules
```

---

# Status

Snort 2 configuration completed successfully.
