---
title: Network Security Hardening and Encryption
difficulty: Intermediate
Cert:
  - Network+
  - NT10-009
Exam-Objectives:
  - Net+ 4.1
  - Net+ 4.3
  - Net+ 4.4
created: 2026-02-13
completed:
time-hours: 4
complete: false
GitHub-ready: false
evidence-files: 13
notes:
tags:
  - security-hardening
  - encryption
  - luks
  - firewall-rules
  - openssl
  - access-control
  - account-security
  - Defense-in-depth
---
---
## Objective Mapping

**4.1 Explain the importance of basic network security concepts**

- Logical security (encryption, access control)
- Defense in depth

**4.3 Explain various activities associated with vulnerability management**

- Hardening techniques

**4.4 Given a scenario, apply network hardening techniques**

- Disable unneeded network services
- Access control lists (ACLs)
- Port security
- Change default credentials

## Scenario

You're a security analyst at a financial institution that just experienced a **security incident**. Forensics revealed that an attacker gained access through:

1. **Unencrypted backup drives** stolen from an unlocked server room
2. **Open firewall ports** (FTP, Telnet) that weren't disabled after migration
3. **Default Guest account** with weak password policy

Your incident response team has contained the breach. Now you must **harden the environment before restoring services**. This lab demonstrates the **defense-in-depth approach** required by compliance frameworks (PCI-DSS, HIPAA, SOC 2):

**Layer 1 (Data)**: Encrypt data at rest using LUKS and OpenSSL  
**Layer 2 (Network)**: Block unused ports, implement ACLs  
**Layer 3 (Identity)**: Enforce password policies, disable risky accounts

This mirrors real SOC workflows where you **systematically eliminate attack vectors** after a security event.

## Learning Objectives

By completing this lab, you will demonstrate ability to:

- **Implement full-disk encryption** using LUKS on Linux partitions
- **Encrypt sensitive files** with OpenSSL symmetric/asymmetric algorithms
- **Block network services** by disabling firewall ports
- **Create ACL rules** to deny traffic from malicious IP ranges
- **Configure password policies** in Active Directory (complexity, expiration)
- **Disable high-risk accounts** (Guest, test accounts)
- **Document hardening baselines** for compliance auditing

## Required Environment

### Hardware/Software

- **VMs**:
    - ACIALMA (Alma Linux 9.3) - Encryption testing
    - ACIDC01 (Windows Server 2022 DC) - Account policies
    - ACIPFSENSE (pfSense 2.7.2) - Firewall hardening
    - ACIWIN11 (Windows 11 Pro) - Testing/validation
- **Tools**:
    - `cryptsetup` (LUKS), `openssl`, pfSense WebGUI, Active Directory Users & Computers

### Host System

- **OS**: Windows 11 Pro or Windows Server 2022
- **RAM**: 12GB minimum
- **Storage**: 20GB free (for LUKS partition creation)
- **Network**: Isolated virtual network (no internet required)

## Deliverables

### Evidence Files (Required)

1. `01-cfdisk-partition-before-luks.png` - Disk layout before encryption
2. `02-luks-format-partition-sda4.txt` - LUKS encryption setup output
3. `03-luks-open-encrypted-partition.txt` - Unlocking LUKS volume
4. `04-mount-encrypted-volume.txt` - Mounted filesystem verification
5. `05-openssl-encrypt-testfile.txt` - File encrypted with AES-256-CBC
6. `06-openssl-decrypt-testfile.txt` - Successfully decrypted file
7. `07-pfsense-port-21-ftp-blocked.png` - FTP port disabled in firewall
8. `08-pfsense-acl-block-malicious-ip.png` - ACL denying 203.0.113.0/24
9. `09-pfsense-firewall-rules-hardened.png` - Final ruleset after hardening
10. `10-ad-password-policy-complexity.png` - Password requirements configured
11. `11-ad-account-lockout-policy.png` - Lockout threshold set to 5 attempts
12. `12-ad-guest-account-disabled.png` - Guest account disabled in ADUC
13. `13-failed-login-attempt-guest.txt` - Verification guest account unusable

### Documentation (Required)

- **Hardening baseline checklist**: Items completed, items pending
- **Encryption key management plan**: Where keys stored, rotation schedule
- **Security controls matrix**: Maps controls to compliance requirements

## Lab Tasks

### Phase 1: Data Encryption (60 minutes)

**Objective**: Protect data at rest using disk and file encryption to prevent data theft if physical media is stolen.

#### Task 1.1: Create LUKS Encrypted Partition

- [ ] **Step 1**: Connect to ACIALMA, open Terminal
    
- [ ] **Step 2**: Launch disk partitioning tool
    
    ```bash
    sudo cfdisk
    # Password: Passw0rd
    ```
    
- [ ] **Step 3**: Resize existing partition to create free space
    
    ```plaintext
    Use arrow keys to select /dev/sda3
    Select [Resize] option
    Change size to: 16.4G
    Press Enter
    
    Select "Free space"
    Select [New]
    Accept default size (creates ~1.9G partition)
    Select [Write]
    Type: yes
    Select [Quit]
    ```
    
- [ ] **Step 4**: Capture evidence
    
    ```bash
    lsblk > ~/01-cfdisk-partition-before-luks.txt
    cat ~/01-cfdisk-partition-before-luks.txt
    ```
    
    - **Screenshot**: Terminal showing partition layout
        - Save as: `01-cfdisk-partition-before-luks.png`

#### Task 1.2: Encrypt Partition with LUKS

- [ ] **Step 5**: Format partition with LUKS
    
    ```bash
    sudo cryptsetup luksFormat /dev/sda4
    
    # WARNING prompt - Type: YES (uppercase)
    # Enter passphrase: SecurePass123!
    # Verify passphrase: SecurePass123!
    ```
    
- [ ] **Step 6**: Capture evidence
    
    ```bash
    sudo cryptsetup luksDump /dev/sda4 > ~/02-luks-format-partition-sda4.txt 2>&1
    cat ~/02-luks-format-partition-sda4.txt
    ```
    
    - **Evidence**: `02-luks-format-partition-sda4.txt`

**Security Note**: LUKS uses AES-256-XTS encryption by default (stronger than BitLocker's AES-128)

#### Task 1.3: Open and Mount Encrypted Volume

- [ ] **Step 7**: Unlock LUKS partition
    
    ```bash
    sudo cryptsetup luksOpen /dev/sda4 encrypted_volume
    # Enter passphrase: SecurePass123!
    
    lsblk | grep encrypted_volume > ~/03-luks-open-encrypted-partition.txt
    cat ~/03-luks-open-encrypted-partition.txt
    ```
    
    - **Evidence**: `03-luks-open-encrypted-partition.txt`
- [ ] **Step 8**: Create filesystem and mount
    
    ```bash
    sudo mkfs.ext4 /dev/mapper/encrypted_volume
    sudo mkdir /mnt/secure_data
    sudo mount /dev/mapper/encrypted_volume /mnt/secure_data
    
    df -h | grep secure_data > ~/04-mount-encrypted-volume.txt
    cat ~/04-mount-encrypted-volume.txt
    ```
    
    - **Evidence**: `04-mount-encrypted-volume.txt`
- [ ] **Step 9**: Test encryption by creating file
    
    ```bash
    echo "Confidential financial data" | sudo tee /mnt/secure_data/testfile.txt
    sudo ls -lh /mnt/secure_data/
    ```
    

**Expected Result**: File accessible only when partition is unlocked. If system reboots, attacker cannot read data without passphrase.

#### Task 1.4: File-Level Encryption with OpenSSL

- [ ] **Step 10**: Create test file
    
    ```bash
    cd ~
    echo "Patient medical records - HIPAA protected" > sensitive.txt
    cat sensitive.txt
    ```
    
- [ ] **Step 11**: Encrypt file with OpenSSL
    
    ```bash
    openssl enc -aes-256-cbc -salt -in sensitive.txt -out sensitive.txt.enc -k "MySecretKey2024"
    
    # Verify encrypted file is unreadable
    cat sensitive.txt.enc > ~/05-openssl-encrypt-testfile.txt
    cat ~/05-openssl-encrypt-testfile.txt
    ```
    
    - **Evidence**: `05-openssl-encrypt-testfile.txt` (shows binary gibberish)
- [ ] **Step 12**: Decrypt file to verify
    
    ```bash
    openssl enc -aes-256-cbc -d -in sensitive.txt.enc -out sensitive_decrypted.txt -k "MySecretKey2024"
    
    cat sensitive_decrypted.txt > ~/06-openssl-decrypt-testfile.txt
    cat ~/06-openssl-decrypt-testfile.txt
    ```
    
    - **Evidence**: `06-openssl-decrypt-testfile.txt` (original plaintext restored)

**Security Note**: In production, use key files instead of passwords: `openssl enc -aes-256-cbc -in file.txt -out file.enc -kfile keyfile.key`

---

### Phase 2: Network Hardening (60 minutes)

**Objective**: Reduce attack surface by blocking unused services and implementing firewall ACLs.

#### Task 2.1: Disable Unused Firewall Ports

- [ ] **Step 1**: Connect to ACIWIN11 → Open Microsoft Edge
    
- [ ] **Step 2**: Navigate to pfSense WebGUI
    
    ```
    URL: http://192.168.0.5
    Username: admin
    Password: Passw0rd
    ```
    
- [ ] **Step 3**: Review current firewall rules
    
    ```
    Firewall → Rules → WAN
    Note any "Pass" rules for legacy protocols:
    - FTP (port 21)
    - Telnet (port 23)
    - HTTP (port 80 if not needed)
    ```
    

#### Task 2.2: Block FTP Port

- [ ] **Step 4**: Create explicit deny rule
    
    ```
    Firewall → Rules → WAN → Add (top of list)
    
    Action: Block
    Protocol: TCP
    Source: Any
    Destination: WAN address
    Destination Port: FTP (21)
    Description: Block insecure FTP - use SFTP instead
    
    Save → Apply Changes
    ```
    
- [ ] **Step 5**: Capture evidence
    
    - **Screenshot**: Firewall rules showing FTP blocked
        - Save as: `07-pfsense-port-21-ftp-blocked.png`

#### Task 2.3: Implement IP-Based ACL

**Scenario**: Threat intelligence identified malicious IP range `203.0.113.0/24` (example range per RFC 5737) scanning your network.

- [ ] **Step 6**: Create ACL deny rule
    
    ```
    Firewall → Rules → WAN → Add (top)
    
    Action: Block
    Protocol: Any
    Source: 203.0.113.0/24
    Destination: Any
    Description: Block malicious IP range - Threat Intel 2024-02-13
    Log: Checked (enable logging)
    
    Save → Apply Changes
    ```
    
- [ ] **Step 7**: Capture evidence
    
    - **Screenshot**: ACL rule blocking malicious subnet
        - Save as: `08-pfsense-acl-block-malicious-ip.png`

#### Task 2.4: Review Final Firewall Posture

- [ ] **Step 8**: Document all hardening rules
    
    ```
    Firewall → Rules → WAN
    
    Verify rule order (top to bottom):
    1. Block 203.0.113.0/24 (malicious IPs)
    2. Block FTP port 21
    3. Block Telnet port 23 (if added)
    4. Allow established/related (stateful firewall)
    5. Default deny all
    ```
    
- [ ] **Step 9**: Capture evidence
    
    - **Screenshot**: Complete hardened ruleset
        - Save as: `09-pfsense-firewall-rules-hardened.png`

**Testing (Optional)**:

```bash
# From external network, verify FTP blocked
telnet 192.168.0.5 21
# Expected: Connection refused or timeout
```

---

### Phase 3: Account Security Hardening (60 minutes)

**Objective**: Prevent unauthorized access through strong password policies and disabled risky accounts.

#### Task 3.1: Configure Password Complexity Policy

- [ ] **Step 1**: Connect to ACIDC01
    
- [ ] **Step 2**: Open Group Policy Management
    
    ```
    Server Manager → Tools → Group Policy Management
    
    Navigate to:
    Forest: aciplab.com
      Domains
        aciplab.com
          Default Domain Policy
    
    Right-click → Edit
    ```
    
- [ ] **Step 3**: Configure password settings
    
    ```
    Computer Configuration
      Policies
        Windows Settings
          Security Settings
            Account Policies
              Password Policy
    
    Set the following:
    - Enforce password history: 24 passwords
    - Maximum password age: 90 days
    - Minimum password age: 1 day
    - Minimum password length: 12 characters
    - Password must meet complexity requirements: Enabled
    - Store passwords using reversible encryption: Disabled
    ```
    
- [ ] **Step 4**: Capture evidence
    
    - **Screenshot**: Password Policy settings configured
        - Save as: `10-ad-password-policy-complexity.png`

**Security Rationale**:

- 12-character minimum resists brute force (78 bits entropy)
- Complexity prevents dictionary attacks
- 90-day rotation limits credential lifespan

#### Task 3.2: Configure Account Lockout Policy

- [ ] **Step 5**: Set lockout threshold
    
    ```
    In Group Policy Editor (same window):
    
    Computer Configuration
      Policies
        Windows Settings
          Security Settings
            Account Policies
              Account Lockout Policy
    
    Configure:
    - Account lockout duration: 30 minutes
    - Account lockout threshold: 5 invalid attempts
    - Reset account lockout counter after: 30 minutes
    ```
    
- [ ] **Step 6**: Capture evidence
    
    - **Screenshot**: Account Lockout Policy settings
        - Save as: `11-ad-account-lockout-policy.png`

**Security Rationale**: 5-attempt threshold balances security (prevents brute force) and usability (allows typos)

#### Task 3.3: Disable Guest Account

- [ ] **Step 7**: Open Active Directory Users & Computers
    
    ```
    Server Manager → Tools → Active Directory Users & Computers
    
    Navigate to:
    aciplab.com
      Users
    
    Right-click Guest account → Disable Account
    ```
    
- [ ] **Step 8**: Capture evidence
    
    - **Screenshot**: Guest account with down arrow (disabled icon)
        - Save as: `12-ad-guest-account-disabled.png`

#### Task 3.4: Verify Account Lockout

- [ ] **Step 9**: Test lockout policy (on ACIWIN11)
    
    ```
    Open Command Prompt:
    
    # Attempt login as Guest 6 times with wrong password
    runas /user:aciplab\Guest cmd
    # Enter wrong password: WrongPass1
    # Repeat 5 more times
    
    # After 5th attempt:
    echo "Account lockout test completed" > C:\Evidence\13-failed-login-attempt-guest.txt
    ```
    
- [ ] **Step 10**: Verify lockout in Event Viewer (ACIDC01)
    
    ```
    Server Manager → Tools → Event Viewer
    Windows Logs → Security
    Filter: Event ID 4740 (Account Lockout)
    
    Verify Guest account locked
    ```
    
- [ ] **Step 11**: Capture evidence
    
    - **Text file**: Login failure message
        - Save as: `13-failed-login-attempt-guest.txt`

---

## Observations & Failure Modes

### Common Issues

#### Issue 1: LUKS Passphrase Forgotten

**Cause**: No passphrase recovery mechanism in LUKS  
**Solution**:

- **Prevention**: Store recovery key in encrypted vault (KeePass, 1Password)
- **Backup**: Create key file: `cryptsetup luksAddKey /dev/sda4 /root/luks-recovery.key`
- **If lost**: Data is unrecoverable (by design - this is feature, not bug)

#### Issue 2: OpenSSL Decryption Returns Garbage

**Cause**: Wrong password or encryption algorithm mismatch  
**Solution**:

```bash
# Verify encryption algorithm used
file sensitive.txt.enc
# Should show: "openssl enc'd data with salted password"

# Try decryption with correct algorithm
openssl enc -aes-256-cbc -d -in file.enc -out file.txt -k "password"
```

#### Issue 3: Firewall Rule Not Blocking Traffic

**Cause**: Rule order - permit rules above deny rules  
**Solution**:

- Firewall processes top-to-bottom
- **Always place deny rules at the TOP**
- Drag-and-drop to reorder rules in pfSense

#### Issue 4: Password Policy Not Applying to Users

**Cause**: Group Policy not refreshed  
**Solution**:

```cmd
# On domain controller
gpupdate /force

# On client workstation
gpupdate /force
gpresult /r
```

#### Issue 5: Account Lockout Triggers for Legitimate Users

**Cause**: Cached credentials on disconnected laptops  
**Solution**:

- Increase threshold to 10 attempts (less secure but more usable)
- Use MFA (reduces brute force risk even with higher threshold)
- Monitor Event ID 4625 (failed logons) for patterns

---

## Analysis & Reflection

### Defense-in-Depth Model

This lab demonstrates **layered security controls**:

```
┌─────────────────────────────────────┐
│ Layer 7: Physical Security          │ ← Server room locks
├─────────────────────────────────────┤
│ Layer 6: Data Encryption (LUKS)     │ ← This lab: Phase 1
├─────────────────────────────────────┤
│ Layer 5: Application Security       │ ← Password policies
├─────────────────────────────────────┤
│ Layer 4: Endpoint Security           │ ← Antivirus, EDR
├─────────────────────────────────────┤
│ Layer 3: Network Security (Firewall)│ ← This lab: Phase 2
├─────────────────────────────────────┤
│ Layer 2: Access Control (IAM)       │ ← This lab: Phase 3
├─────────────────────────────────────┤
│ Layer 1: Perimeter Security          │ ← IDS/IPS, firewalls
└─────────────────────────────────────┘
```

**Key Principle**: If attacker bypasses one layer, other layers still protect assets.

### Encryption Best Practices

**LUKS (Full Disk Encryption)**:

- **Use Case**: Protect laptops, backup drives, cloud VM disks
- **Performance**: ~5-10% CPU overhead (negligible on modern hardware)
- **Unlock Methods**:
    - Passphrase (what you know)
    - Key file (what you have)
    - TPM chip (hardware-bound)

**OpenSSL (File Encryption)**:

- **Symmetric (AES-256)**: Fast, single shared key
    - Use for: Backup archives, personal files
- **Asymmetric (RSA)**: Slow, public/private key pair
    - Use for: Email encryption (PGP), secure key exchange

**Key Management Hierarchy**:

```
Master Key (stored in TPM/HSM)
  └─> Data Encryption Key (DEK)
      └─> Actual encrypted data

Rotation: Change DEK annually, re-encrypt master key only
```

### Firewall Hardening Checklist

**Services to Disable (Legacy Protocols)**:

- [x] FTP (port 21) → Use SFTP (port 22)
- [x] Telnet (port 23) → Use SSH (port 22)
- [ ] HTTP (port 80) → Use HTTPS (port 443)
- [ ] SNMP v1/v2 (port 161) → Use SNMPv3 (encrypted)
- [ ] SMBv1 (port 445) → Use SMBv3 (signing required)

**ACL Strategy**:

1. **Geo-blocking**: Block countries not in business scope
2. **Threat Intel Feeds**: Auto-update malicious IP lists (Alienvault OTX, Abuse.ch)
3. **Rate Limiting**: Block sources with >100 connections/minute

### Password Policy Trade-offs

|Requirement|Security Benefit|Usability Impact|
|---|---|---|
|12-char minimum|High (78-bit entropy)|Low (memorable with passphrase)|
|90-day rotation|Medium (limits breach window)|Medium (users write down passwords)|
|Complexity|Medium (prevents dictionary)|High (users add "!" to end)|
|**Recommendation**|Use 16-char passphrases + MFA|Better than complex 12-char|

**Modern Approach (NIST SP 800-63B)**:

- **Eliminate** periodic rotation (causes weak passwords)
- **Require** 15+ characters (passphrase)
- **Mandate** MFA for all privileged accounts

### Real-World Applications

**SOC Analyst Perspective**:

- **LUKS Detection**: Monitor for mass `cryptsetup` usage (ransomware uses LUKS to encrypt victim drives)
- **Firewall Logs**: Alert on blocked traffic spikes (DDoS indicator)
- **Account Lockouts**: Event ID 4740 correlation = brute force attempt

**Incident Response Workflow**:

1. **Containment**: Block malicious IPs (ACLs)
2. **Eradication**: Disable compromised accounts
3. **Recovery**: Decrypt backups (LUKS/OpenSSL)
4. **Post-Incident**: Implement password policies to prevent reinfection

**Compliance Mapping**:

```
PCI-DSS Requirement 8.2.3 → Password complexity (Phase 3)
HIPAA § 164.312(a)(2)(iv) → Encryption of ePHI (Phase 1)
SOC 2 CC6.1 → Logical access controls (Phase 2 & 3)
```

### Time Tracking

|Phase|Estimated|Actual|Notes|
|---|---|---|---|
|Phase 1: Encryption|60 min|___ min||
|Phase 2: Network Hardening|60 min|___ min||
|Phase 3: Account Security|60 min|___ min||
|**Total**|**180 min**|**___ min**||

---

## Exam Relevance Notes

### Exam Scenario Mapping

**N10-009 Performance-Based Questions (PBQs)**:

- **"Harden a network after a breach"**
    - Given: Firewall ruleset with FTP/Telnet open
    - Action: Create deny rules, reorder to top
- **"Configure password policy for compliance"**
    - Given: Current policy (8-char, no complexity)
    - Action: Set to 12-char, complexity, 90-day rotation

**Multiple Choice Question Patterns**:

- "Which encryption standard is used by LUKS?" → AES-256-XTS
- "What is the purpose of an account lockout policy?" → Prevent brute force attacks
- "Which port should be blocked to prevent insecure file transfers?" → 21 (FTP)

### Key Exam Concepts Reinforced

**Encryption Terms**:

- **Encryption at Rest**: LUKS, BitLocker, FileVault
- **Encryption in Transit**: TLS, IPSec, SSH
- **Symmetric**: Single key (AES, DES, 3DES)
- **Asymmetric**: Public/private key pair (RSA, ECC)

**Hardening Techniques**:

- **Least Privilege**: Users get minimum necessary permissions
- **Defense in Depth**: Multiple security layers
- **Disable Unused Services**: Reduces attack surface
- **Strong Authentication**: Complex passwords + MFA

**Access Control Models**:

- **DAC** (Discretionary): Owner controls access (NTFS permissions)
- **MAC** (Mandatory): System enforces labels (classified networks)
- **RBAC** (Role-Based): Access by job role (AD groups)

---

## Submission Checklist

Before marking this lab complete, verify:

- [ ] All 13 evidence files collected and named correctly
- [ ] Screenshots show encrypted partition, firewall rules, AD policies
- [ ] Hardening baseline checklist completed
- [ ] Encryption key management plan documented
- [ ] Security controls matrix maps to compliance requirements
- [ ] Time tracking completed
- [ ] Evidence files moved to `/mnt/user-data/outputs`

**Final Evidence Count**: 13 files  
**Documentation Pages**: 1 (this lab markdown)  
**Ready for GitHub**: Yes / No

---

## Next Steps

After completing this lab, you should:

1. **Review**: Study NIST Cybersecurity Framework (CSF) 2.0 control families
2. **Practice**: Implement CIS Benchmarks for Windows Server 2022
3. **Advance**: Proceed to Lab 08 (Network Troubleshooting & Wireless)
4. **Optional Challenge**: Set up automated compliance scanning (OpenSCAP, Lynis)

**Additional Resources**:

- [NIST SP 800-53 Security Controls](https://csrc.nist.gov/publications/detail/sp/800-53/rev-5/final)
- [CIS Benchmarks](https://www.cisecurity.org/cis-benchmarks/)
- [LUKS Documentation](https://gitlab.com/cryptsetup/cryptsetup)
- CompTIA Network+ N10-009 Official Study Guide, Chapter 4 (Security)

---

## Appendix: Hardening Baseline Checklist

**Encryption Controls**:

- [x] Full-disk encryption on servers (LUKS/BitLocker)
- [x] File encryption for backups (OpenSSL/GPG)
- [ ] Database encryption at rest (TDE - Transparent Data Encryption)
- [ ] Email encryption (S/MIME or PGP)

**Network Controls**:

- [x] Disabled FTP (use SFTP)
- [x] Disabled Telnet (use SSH)
- [x] IP-based ACLs for threat intel
- [ ] Geo-blocking for non-business countries
- [ ] IDS/IPS signatures updated

**Identity Controls**:

- [x] Password complexity requirements
- [x] Account lockout policy
- [x] Guest account disabled
- [ ] MFA for privileged accounts
- [ ] Service accounts use managed service accounts (gMSA)
- [ ] Privileged Access Management (PAM) solution deployed

**Monitoring Controls**:

- [ ] SIEM ingesting firewall logs
- [ ] Failed login alerts (Event ID 4625)
- [ ] Account lockout alerts (Event ID 4740)
- [ ] Encryption key access auditing

**Status**: 6/19 controls implemented (32% complete)  
**Next Priority**: Deploy MFA for admin accounts