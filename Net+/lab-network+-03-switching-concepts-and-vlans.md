---
title: "Switching Concepts: VLANs and MTU Configuration"
difficulty: Intermediate
Cert:
  - Network+
  - NT10-009
Exam-Objectives:
  - Net+ 1.2
  - Net+ 2.2
created: 2026-02-13
completed:
time-hours: 2
complete: false
GitHub-ready: false
evidence-files: 12
notes:
tags:
  - switching
  - vlans
  - mtu
  - hyper-v
  - tiny-core-linux
  - layer-2
---
---
## Objective Mapping

**2.2 Given a scenario, configure switching technologies and features**

- Virtual Local Area Network (VLAN)
- Interface configuration
- Maximum Transmission Unit (MTU)

**1.2 Explain the characteristics of network topologies and network types**

- LAN (Local Area Network)

## Scenario

You're a network engineer at a healthcare organization preparing for a compliance audit. The auditor requires **network segmentation** between departments (medical records, billing, general staff) to meet HIPAA regulations. Your team has decided to implement **VLANs** to logically separate traffic without purchasing additional physical switches.

Additionally, the VoIP vendor reports intermittent call quality issues. After analysis, you suspect **MTU fragmentation** is causing packet loss on certain network segments. Your task is to:

1. **Demonstrate VLAN isolation** using Hyper-V virtual machines
2. **Test MTU settings** and verify how oversized packets are handled

This lab simulates a common SOC/network admin scenario: **using Layer 2 features (VLANs, MTU) to improve security and performance** without changing physical infrastructure.

## Learning Objectives

By completing this lab, you will demonstrate ability to:

- **Create and configure VLANs** in a virtualized environment
- **Isolate traffic** between virtual machines using VLAN segmentation
- **Test inter-VLAN communication** to verify isolation
- **Modify MTU settings** on network interfaces
- **Troubleshoot MTU-related fragmentation** using ping tests
- **Document VLAN design** with proper naming conventions

## Required Environment

### Hardware/Software

- **Hypervisor**: Hyper-V on Windows Server 2022 (ACIDM01)
- **Guest VMs**:
    - Windows Server 2022 (Member Server) – ACIDM01 (Hyper-V host)
    - Tiny Core Linux VM1 (VLAN 10 – created during lab)
    - Tiny Core Linux VM2 (VLAN 20 – created during lab)
- **Tools**:
    - Hyper-V Manager
    - Virtual Switch Manager
    - Tiny Core Linux (lightweight VM for testing)

### Host System

- **OS**: Windows Server 2022 or Windows 11 Pro with Hyper-V enabled
- **RAM**: 12GB minimum (VMs require minimal resources)
- **Storage**: 10GB free (Tiny Core Linux is ~50MB)
- **Network**: Internal virtual switch (no external access required)

## Deliverables

### Evidence Files (Required)

1. `01-virtual-switch-mod9switch-creation.png` – Internal virtual switch configuration
2. `02-vm1-creation-wizard-summary.png` – VM1 attached to Mod9Switch
3. `03-vm2-creation-wizard-summary.png` – VM2 attached to Mod9Switch
4. `04-vm1-vlan-10-configuration.png` – VM1 network adapter set to VLAN 10
5. `05-vm2-vlan-20-configuration.png` – VM2 network adapter set to VLAN 20
6. `06-vm1-ifconfig-output.txt` – IP configuration of VM1 (192.168.10.10)
7. `07-vm2-ifconfig-output.txt` – IP configuration of VM2 (192.168.20.10)
8. `08-vm1-ping-vm2-vlan-isolation-test.txt` – Ping failure showing VLAN isolation
9. `09-vm1-mtu-default-1500.txt` – Default MTU setting
10. `10-vm1-ping-mtu-1472-success.png` – Successful ping with MTU 1472 (-f -l 1472)
11. `11-vm1-ping-mtu-1473-failure.png` – Failed ping with MTU 1473 (fragmentation)
12. `12-vm2-mtu-modified-9000.txt` – Jumbo frames MTU configuration

### Documentation (Required)

- **VLAN design document**: Which departments go in which VLANs and why
- **MTU testing matrix**: Results of ping tests at different MTU sizes
- **Troubleshooting log**: Issues encountered and resolutions

## Lab Tasks

### Phase 1: VLAN Configuration and Isolation Testing

**Estimated Time**: 70 minutes

**Objective**: Create two VMs on different VLANs and verify they cannot communicate due to VLAN isolation.

#### Task 1.1: Create Internal Virtual Switch

- [ ] **Step 1**: Open Hyper-V Manager on ACIDM01
    
    ```plaintext
    Taskbar → Hyper-V Manager
    Actions pane → Virtual Switch Manager
    ```
    
- [ ] **Step 2**: Create internal switch
    
    ```plaintext
    Select: "Internal" virtual switch type
    Click: "Create Virtual Switch"
    Name: Mod9Switch
    Connection type: Internal network
    Notes: "VLAN demonstration switch"
    Click OK
    ```
    
- [ ] **Step 3**: Capture evidence
    
    - **Screenshot**: Virtual Switch Manager showing Mod9Switch
        - Save as: `01-virtual-switch-mod9switch-creation.png`

#### Task 1.2: Create VM1 (VLAN 10)

- [ ] **Step 4**: Create new virtual machine
    
    ```plaintext
    Actions → New → Virtual Machine
    
    Wizard settings:
    - Name: VM1
    - Generation: Generation 1
    - Memory: 512MB (default)
    - Network: Mod9Switch
    - Virtual Hard Disk: Create new (dynamically expanding, 8GB)
    - Installation Options: Install from ISO image
      - Browse to: C:\Labs\Module_9\Core-current.iso
    ```
    
- [ ] **Step 5**: Complete VM creation
    
    ```plaintext
    Click: Finish
    Do NOT start VM yet
    ```
    
- [ ] **Step 6**: Capture evidence
    
    - **Screenshot**: VM1 summary in Hyper-V Manager
        - Save as: `02-vm1-creation-wizard-summary.png`

#### Task 1.3: Assign VLAN 10 to VM1

- [ ] **Step 7**: Configure VLAN tagging
    
    ```plaintext
    Right-click VM1 → Settings
    Network Adapter → Expand
    Select: "Advanced Features"
    
    VLAN section:
    - Check: "Enable virtual LAN identification"
    - VLAN ID: 10
    
    Click Apply → OK
    ```
    
- [ ] **Step 8**: Capture evidence
    
    - **Screenshot**: VM1 Network Adapter showing VLAN 10
        - Save as: `04-vm1-vlan-10-configuration.png`

#### Task 1.4: Create VM2 (VLAN 20)

- [ ] **Step 9**: Create second VM
    
    ```plaintext
    Repeat VM creation process:
    - Name: VM2
    - Same settings as VM1
    - Connected to: Mod9Switch
    - ISO: Core-current.iso
    ```
    
- [ ] **Step 10**: Capture evidence
    
    - **Screenshot**: VM2 summary
        - Save as: `03-vm2-creation-wizard-summary.png`
- [ ] **Step 11**: Assign VLAN 20
    
    ```plaintext
    VM2 → Settings → Network Adapter → Advanced Features
    VLAN ID: 20
    Apply → OK
    ```
    
- [ ] **Step 12**: Capture evidence
    
    - **Screenshot**: VM2 Network Adapter showing VLAN 20
        - Save as: `05-vm2-vlan-20-configuration.png`

#### Task 1.5: Configure IP Addresses

- [ ] **Step 13**: Start VM1
    
    ```plaintext
    Right-click VM1 → Connect
    Start VM
    Wait for Tiny Core Linux boot
    ```
    
- [ ] **Step 14**: Assign static IP to VM1
    
    ```bash
    # In VM1 terminal (Tiny Core auto-logs in)
    sudo ifconfig eth0 192.168.10.10 netmask 255.255.255.0 up
    ifconfig eth0 > ~/vm1-ifconfig.txt
    cat ~/vm1-ifconfig.txt
    ```
    
- [ ] **Step 15**: Export evidence from VM1
    
    ```plaintext
    # Method 1: Screenshot
    Screenshot terminal output → 06-vm1-ifconfig-output.txt
    
    # Method 2: If shared folder available
    Copy ~/vm1-ifconfig.txt to host
    ```
    
- [ ] **Step 16**: Configure VM2
    
    ```bash
    # Start VM2, connect to console
    sudo ifconfig eth0 192.168.20.10 netmask 255.255.255.0 up
    ifconfig eth0 > ~/vm2-ifconfig.txt
    cat ~/vm2-ifconfig.txt
    ```
    
- [ ] **Step 17**: Export evidence from VM2
    
    - **Screenshot/Text File**: VM2 ifconfig output
        - Save as: `07-vm2-ifconfig-output.txt`

#### Task 1.6: Test VLAN Isolation

- [ ] **Step 18**: Attempt ping from VM1 to VM2
    
    ```bash
    # On VM1
    ping -c 4 192.168.20.10 > ~/vlan-isolation-test.txt
    cat ~/vlan-isolation-test.txt
    ```
    
    **Expected Result**: Ping should **fail** due to VLAN isolation
    
- [ ] **Step 19**: Capture evidence
    
    - **Screenshot/Text File**: Ping failure output
        - Save as: `08-vm1-ping-vm2-vlan-isolation-test.txt`
- [ ] **Step 20**: Verify isolation rationale
    
    ```markdown
    ## VLAN Isolation Test Results
    
    VM1 (VLAN 10) → VM2 (VLAN 20): FAIL ✓ (Expected)
    Reason: Different VLANs cannot communicate without router/Layer 3 device
    
    If ping succeeded: VLAN tagging not applied correctly
    ```
    

**Expected Result**: VMs on different VLANs cannot ping each other, demonstrating Layer 2 segmentation.

---

### Phase 2: MTU Configuration and Testing

**Estimated Time**: 50 minutes

**Objective**: Modify MTU settings and observe fragmentation behavior using ping tests.

#### Task 2.1: Verify Default MTU

- [ ] **Step 1**: Check default MTU on VM1
    
    ```bash
    # On VM1 (Tiny Core Linux)
    ifconfig eth0 | grep MTU
    
    # Or more detailed:
    ip link show eth0 | grep mtu
    ```
    
    **Expected Output**: `MTU:1500` (standard Ethernet MTU)
    
- [ ] **Step 2**: Capture evidence
    
    ```bash
    ifconfig eth0 | grep MTU > ~/mtu-default.txt
    cat ~/mtu-default.txt
    ```
    
    - **Screenshot/Text**: Default MTU output
        - Save as: `09-vm1-mtu-default-1500.txt`

#### Task 2.2: Test MTU Fragmentation (Windows Host)

**Note**: This task is performed on the **Hyper-V host (ACIDM01)**, not inside VMs, as Tiny Core Linux ping doesn't support `-f` flag.

- [ ] **Step 3**: Open Command Prompt on ACIDM01
    
    ```cmd
    # Find Mod9Switch adapter IP (assigned when internal switch created)
    ipconfig /all | findstr /C:"vEthernet (Mod9Switch)"
    
    # Typical IP: 192.168.x.1 (auto-assigned)
    ```
    
- [ ] **Step 4**: Test maximum MTU without fragmentation
    
    ```cmd
    # Ping with 1472 bytes data + 28 bytes header = 1500 bytes (MTU)
    ping -f -l 1472 192.168.10.10
    
    # -f = Don't Fragment flag
    # -l 1472 = Send 1472 bytes of data
    ```
    
    **Expected Result**: Success (1472 + 28 = 1500, fits in MTU)
    
- [ ] **Step 5**: Capture evidence
    
    - **Screenshot**: Successful ping with MTU 1472
        - Save as: `10-vm1-ping-mtu-1472-success.png`
- [ ] **Step 6**: Test MTU fragmentation threshold
    
    ```cmd
    # Attempt 1473 bytes data (requires 1501 byte frame)
    ping -f -l 1473 192.168.10.10
    ```
    
    **Expected Result**: "Packet needs to be fragmented but DF set" error
    
- [ ] **Step 7**: Capture evidence
    
    - **Screenshot**: Failed ping with MTU 1473 error
        - Save as: `11-vm1-ping-mtu-1473-failure.png`

#### Task 2.3: Configure Jumbo Frames

- [ ] **Step 8**: Modify MTU on VM2
    
    ```bash
    # On VM2 terminal
    sudo ifconfig eth0 mtu 9000
    ifconfig eth0 | grep MTU > ~/mtu-jumbo-frames.txt
    cat ~/mtu-jumbo-frames.txt
    ```
    
    **Expected Output**: `MTU:9000` (Jumbo frames enabled)
    
- [ ] **Step 9**: Capture evidence
    
    - **Screenshot/Text**: VM2 with MTU 9000
        - Save as: `12-vm2-mtu-modified-9000.txt`
- [ ] **Step 10**: Test jumbo frame limitation
    
    ```bash
    # On VM2, attempt to ping VM1 with large packet
    ping -c 1 -s 8972 192.168.10.10
    
    # -s 8972 = 8972 bytes data + 28 bytes header = 9000 bytes
    ```
    
    **Note**: Ping may fail if Mod9Switch doesn't support jumbo frames in virtual environment
    

#### Task 2.4: Document MTU Testing Matrix

- [ ] **Step 11**: Create MTU testing summary
    
    ```markdown
    ## MTU Testing Results| Packet Size (Data) | Total Frame Size | Result | Notes ||--------------------|------------------|--------|-------|| 1472 bytes | 1500 bytes | Success | Fits standard MTU || 1473 bytes | 1501 bytes | Failure | DF flag prevents fragmentation || 8972 bytes | 9000 bytes | Varies | Depends on virtual switch support |**Key Takeaway**: MTU mismatches cause:- Fragmentation (if DF not set)- Packet drops (if DF set)- Performance degradation
    ```
    

**Expected Result**: Understanding of MTU impact on network performance and fragmentation.

---

## Observations & Failure Modes

### Common Issues

#### Issue 1: VMs on Different VLANs Can Communicate

**Cause**: VLAN tagging not enabled or wrong VLAN ID assigned  
**Solution**:

```plaintext
1. Verify VLAN ID in VM settings:
   VM → Settings → Network Adapter → Advanced Features
   Ensure "Enable virtual LAN identification" is CHECKED
   Verify VLAN ID matches requirement (10, 20, etc.)

2. Restart VM after changing VLAN settings
3. Check virtual switch type (must be Internal or External, not Private)
```

#### Issue 2: Tiny Core Linux ISO Not Booting

**Cause**: ISO file corrupted or wrong architecture  
**Solution**:

- Download fresh Tiny Core Linux ISO (Core-current.iso)
- Verify MD5 checksum
- Use 64-bit ISO if available
- Alternative: Use Ubuntu Server minimal ISO

#### Issue 3: Cannot Ping 192.168.10.10 from ACIDM01

**Cause**: Hyper-V internal switch doesn't route to VLANs by default  
**Solution**:

```cmd
# On ACIDM01, assign IP in same subnet as target VLAN
netsh interface ip set address "vEthernet (Mod9Switch)" static 192.168.10.1 255.255.255.0

# Then test ping
ping 192.168.10.10
```

#### Issue 4: MTU Test Shows Success with 1473 Bytes

**Cause**: DF flag not set or MTU auto-adjusted by OS  
**Solution**:

```cmd
# Ensure -f flag is used (Windows)
ping -f -l 1473 192.168.10.10

# Verify MTU on interface
netsh interface ipv4 show subinterfaces
```

---

## Analysis & Reflection

### VLAN Design Best Practices

**Healthcare Example (HIPAA Compliance)**:

```
VLAN 10 - Medical Records (Protected Health Information)
  - Isolated from general staff
  - ACLs restrict access to authorized systems only
  
VLAN 20 - Billing Department
  - Payment card data (PCI-DSS compliance)
  - Separate from clinical systems
  
VLAN 30 - General Staff / Guest WiFi
  - No access to sensitive VLANs
  - Internet access only
  
VLAN 99 - Management (Native VLAN)
  - Switch management interfaces
  - Jump hosts for administrators
```

**VLAN Naming Conventions**:

- Use descriptive names: `VLAN_MEDICAL` not `VLAN10`
- Document purpose in network diagrams
- Tag native VLAN (often VLAN 1 or 99) for management

### MTU Considerations

**When to Modify MTU**:

- **Increase to 9000 (Jumbo Frames)**: SANs, backup networks, VM migration
- **Decrease below 1500**: VPN tunnels (IPSec adds 50-60 bytes overhead)
- **Keep at 1500**: General internet traffic

**Troubleshooting MTU Issues**:

```bash
# Linux: Test path MTU discovery
tracepath example.com

# Windows: Test with incrementing packet sizes
ping -f -l 1400 example.com
ping -f -l 1450 example.com
ping -f -l 1472 example.com
```

### Security Implications

**VLAN Hopping Attacks**:

- **Double Tagging**: Attacker sends frame with two VLAN tags
- **Switch Spoofing**: Attacker emulates trunk port
- **Mitigation**:
    - Disable DTP (Dynamic Trunking Protocol)
    - Set unused ports to access mode
    - Use non-default native VLAN

**MTU as Attack Vector**:

- **MTU Black Hole**: Firewall blocks ICMP "Fragmentation Needed" messages
- **Solution**: Enable Path MTU Discovery (PMTUD)

### Real-World Applications

**SOC Analyst Perspective**:

- VLAN isolation prevents lateral movement in ransomware attacks
- Monitor for VLAN hopping attempts in IDS logs
- Verify critical systems are on isolated VLANs

**Network Admin Perspective**:

- VLANs reduce broadcast domains (improves performance)
- MTU tuning can improve iSCSI SAN throughput by 30%
- Document VLAN assignments in network topology diagrams

### Time Tracking

|Phase|Estimated|Actual|Notes|
|---|---|---|---|
|Phase 1: VLANs|70 min|___ min||
|Phase 2: MTU|50 min|___ min||
|**Total**|**120 min**|**___ min**||

---

## Exam Relevance Notes

### Exam Scenario Mapping

**N10-009 Performance-Based Questions (PBQs)**:

- **Configure VLAN on switch port**
    - Given: Network diagram, requirement to isolate HR department
    - Action: Set port VLAN ID to 50
- **Troubleshoot MTU mismatch**
    - Symptom: Large file transfers fail, small files work
    - Solution: Check MTU on both ends, adjust to match

**Multiple Choice Question Patterns**:

- "A company wants to segment traffic without buying new switches. Which technology should they use?"
    - Answer: VLANs
- "What is the standard Ethernet MTU size?"
    - Answer: 1500 bytes
- "Jumbo frames have an MTU of:"
    - Answer: 9000 bytes

### Key Exam Concepts Reinforced

**VLAN Fundamentals**:

- **Purpose**: Logical segmentation at Layer 2
- **Tagging Protocol**: IEEE 802.1Q (adds 4-byte tag)
- **Trunking**: Carries multiple VLANs between switches
- **Native VLAN**: Untagged traffic on trunk port

**MTU Concepts**:

- **Path MTU Discovery (PMTUD)**: Finds largest MTU along path
- **Fragmentation**: Breaks packets larger than MTU
- **DF (Don't Fragment) Bit**: Prevents fragmentation, triggers ICMP error
- **Jumbo Frames**: MTU > 1500, requires switch support

**Layer 2 vs. Layer 3**:

- VLANs operate at **Layer 2** (Data Link)
- Inter-VLAN routing requires **Layer 3** device (router/L3 switch)
- Exam trap: "Can VLANs communicate without a router?" → NO

---

## Submission Checklist

Before marking this lab complete, verify:

- [ ] All 12 evidence files collected and named correctly
- [ ] Screenshots show VLAN IDs clearly visible in Hyper-V settings
- [ ] MTU testing matrix completed with results
- [ ] VLAN design document includes department assignments
- [ ] Troubleshooting log documents at least one issue
- [ ] Time tracking completed
- [ ] Evidence files moved to `/mnt/user-data/outputs`

**Final Evidence Count**: 12 files  
**Documentation Pages**: 1 (this lab markdown)  
**Ready for GitHub**: Yes / No

---

## Next Steps

After completing this lab, you should:

1. **Review**: Study inter-VLAN routing (router-on-a-stick vs. Layer 3 switch)
2. **Practice**: Build a multi-VLAN network with pfSense routing between VLANs
3. **Advance**: Proceed to Lab 04 (IPv4 Addressing) to assign subnets per VLAN
4. **Optional Challenge**: Configure VLAN trunking between two Hyper-V switches

**Additional Resources**:

- [Cisco: VLAN Configuration Guide](https://www.cisco.com/c/en/us/td/docs/switches/lan/catalyst9300/software/release/16-12/configuration_guide/vlan/b_1612_vlan_9300_cg.html)
- CompTIA Network+ N10-009 Official Study Guide, Chapter 2 (Switching)
- [Understanding MTU and PMTUD](https://packetlife.net/blog/2008/aug/18/path-mtu-discovery/)