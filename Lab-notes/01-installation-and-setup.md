# 01 - Installation and Setup: Installing Snort 2 on Kali Linux

## Project Context

This section documents the steps I followed to successfully install Snort 2.9.20 on my Kali Linux VM from source. I initially had Snort 3 installed, but I decided to revert to Snort 2 because Snort 2 is widely used for learning IDS/IPS fundamentals, especially custom rule writing and basic detection engineering.

## Lab Environment

| Item | Details |
|---|---|
| Operating System | Kali Linux |
| Tool Installed | Snort |
| Version Installed | Snort 2.9.20 GRE Build 82 |
| Installation Method | Manual installation from source |
| Purpose | IDS/IPS lab for SOC analyst practice |

---

## Step 1: Removed Existing Snort Installation

I first removed the existing Snort installation from the system.

```bash
sudo apt remove snort -y
sudo apt autoremove -y
```
## Step 2: Installed Required Dependencies

I installed the required build tools and libraries needed to compile Snort from source.

```bash
sudo apt update
sudo apt install -y build-essential libpcap-dev libpcre3-dev libdumbnet-dev bison flex zlib1g-dev liblzma-dev openssl libssl-dev`bash
```
## Step 3: Downloaded Snort 2 Source Code

I downloaded Snort 2.9.20 from the official Snort download source.

```bash
wget https://www.snort.org/downloads/snort/snort-2.9.20.tar.gz
```

Then I extracted the archive:

```bash
tar -xvzf snort-2.9.20.tar.gz
cd snort-2.9.20
```

## Step 4: First Configuration Attempt and DAQ Error

When I first ran:

```bash
./configure
```

I received this error:

```
ERROR! daq_static library not found, go get it from http://www.snort.org/.
```

This happened because Snort 2 requires the DAQ library before it can compile successfully.

## Step 5: Installed DAQ

I installed DAQ dependencies:

```bash
sudo apt update
sudo apt install -y build-essential libpcap-dev libtool automake autoconf
```

Then I downloaded DAQ:
```bash
wget https://www.snort.org/downloads/snort/daq-2.0.7.tar.gz
```

Extracted it:

```bash
tar -xvzf daq-2.0.7.tar.gz
cd daq-2.0.7
```

Configured, compiled, and installed DAQ:

```bash
./configure
make
sudo make install
sudo ldconfig
```

## Step 6: Returned to Snort Source Directory

After installing DAQ, I returned to the Snort source directory:

```bash
cd ~/snort-2.9.20
```

## Step 7: LuaJIT / OpenAppID Error

When I tried configuring Snort again, I encountered this error:

```
ERROR! LuaJIT library not found. Go get it from http://www.luajit.org/
Try compiling without openAppId using '--disable-open-appid'
configure: error: "Fatal!"
```

To solve this, I disabled OpenAppID because it was not required for my basic IDS/IPS lab.

```bash
./configure --disable-open-appid
```

## Step 8: RPC Header Error During Compilation

After running:

```bash
make
```

I received this error:

```bash
fatal error: rpc/rpc.h: No such file or directory
```

This meant the RPC development headers were missing.

I installed the required package:

```bash
sudo apt install -y libtirpc-dev
```

Then I cleaned and reconfigured Snort with the correct RPC header path:

```bash
make clean
./configure --disable-open-appid CFLAGS="-I/usr/include/tirpc" LDFLAGS="-ltirpc"
```

## Step 9: Fixed PDF Decompression Function Error

During compilation, I encountered anoter error:

```
error: too many arguments to function ‘File_Decomp_PDF’; expected 0, have 1
```

To fix it, I patched the Snort header file:

```bash
sed -i 's/fd_status_t File_Decomp_PDF();/fd_status_t File_Decomp_PDF(File_Decomp_Session_t *SessionPtr);/' src/preprocessors/HttpInspect/include/file_decomp_PDF.h
```

This caused another type error because File_Decomp_Session_t was not recognized.

I then changed the declaration to use the correct function type:

```bash
sed -i 's/fd_status_t File_Decomp_PDF(File_Decomp_Session_t \*SessionPtr);/fd_status_t File_Decomp_PDF(void *SessionPtr);/' src/preprocessors/HttpInspect/include/file_decomp_PDF.h
```

This created a conflicting type error, so I corrected it again using the proper type from the source code:

```bash
sed -i 's/fd_status_t File_Decomp_PDF(void \*SessionPtr);/fd_status_t File_Decomp_PDF(fd_session_p_t SessionPtr);/' src/preprocessors/HttpInspect/include/file_decomp_PDF.h
```

## Step 10: Fixed DCE2 Reload Safe Memcap Error

During another compilation attempt, I received this error:

```
error: too many arguments to function ‘DCE2_GetReloadSafeMemcap’; expected 0, have 1
```

The function declaration did not match how the function was being called later in the source file.

I patched the declaration:

```bash
sed -i 's/static uint32_t DCE2_GetReloadSafeMemcap();/static uint32_t DCE2_GetReloadSafeMemcap(tSfPolicyUserContextId pConfig);/' src/dynamic-preprocessors/dcerpc2/spp_dce2.c
```

## Step 11: Successfully Compiled Snort

After applying the patches, I ran:

```bash
make
```

This time, the compilation completed successfully.

Step 12: Installed Snort

I installed Snort using:

```bash
sudo make install
```

Then I refreshed the shared library cache:

```bash
sudo ldconfig
```

## Step 13: Verified Snort Installation

I verified the installation using:

```bash
Step 13: Verified Snort Installation

I verified the installation using:
```

The output confirmed that Snort 2.9.20 was successfully installed:

```
,,_     -*> Snort! <*-
o"  )~   Version 2.9.20 GRE (Build 82)
''''    By Martin Roesch & The Snort Team
        Using libpcap version 1.10.6
        Using PCRE version: 8.39
        Using ZLIB version: 1.3.1
```

I also tested the normal command:

```bash
snort -V
```

This also worked successfully, confirming that Snort was available from the command line.

## Final Installation Result

Snort 2 was successfully installed on my Kali Linux  VM.
```
Version 2.9.20 GRE (Build 82)
```

## Challenges Faced

During the installation, I encountered and resolved the following issues:

1. Missing DAQ library
2. Missing LuaJIT dependency
3. OpenAppID configuration issue
4. Missing RPC header file
5. Source-code compatibility issue in file_decomp_PDF.h
6, Source-code compatibility issue in spp_dce2.c

## Key Lessons Learned

This installation helped me understand that building security tools from source often requires troubleshooting missing dependencies, compiler errors, and compatibility issues. I also learned how to read error messages carefully, install missing development libraries, patch source files, and verify successful installation.




