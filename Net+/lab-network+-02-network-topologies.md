---
title: "Network Topologies: Star and Full Mesh Implementation"
difficulty: Foundation → Intermediate
Cert:
  - Network+
  - NT10-009
Exam-Objectives:
created: 2026-02-13
completed:
time-hours: 3
complete: false
GitHub-ready: false
evidence-files: 13
notes:
tags:
  - network-topologies
  - network-plus-nt0-009
  - hyper-v
  - virtual-networking
  - star-topology
  - full-mesh-topology
  - nic-teaming
---
---
---

lab_title: "Network Topologies: Star and Full Mesh Implementation" module: "Module 1: Network Fundamentals" difficulty: "Foundation → Intermediate" estimated_time: "150 minutes" tags:

- network-plus-n10-009
- network-topologies
- hyper-v
- virtual-networking
- star-topology
- full-mesh-topology
- nic-teaming
- intermediate status: "Active"

---

## Associated Exam

**CompTIA Network+ N10-009**

## Objective Mapping

**1.2 Explain the characteristics of network topologies and network types**

- Mesh
- Star/hub-and-spoke
- Hybrid

**2.1 Compare and contrast various devices, their features, and their appropriate placement on the network**

- Switch (used in star topology)
- Virtual network components

**1.7 Explain basic corporate and datacenter network architecture**

- Software-defined networking (SDN)
- Virtual network components (virtual switches, NICs)

## Scenario

You're a systems administrator at a mid-sized financial services firm migrating from physical infrastructure to virtualized environments. Your manager has tasked you with **demonstrating two critical network topologies** using Windows Server 2022 Hyper-V before deploying them in production.

**Business Context**: The IT director needs to understand the trade-offs between **star topology** (simple, centralized) and **full mesh topology** (redundant, high-bandwidth) for two upcoming projects:

1. **Branch office setup** (20 users) → Star topology for cost efficiency
2. **High-availability cluster** (3 database servers) → Full mesh for redundancy

Your deliverable is a **working proof-of-concept** showing both topologies in Hyper-V, complete with documentation explaining when to use each design. This lab simulates the type of architecture planning SOC analysts and network engineers perform when designing resilient, segmented networks.

## Learning Objectives

By completing this lab, you will demonstrate ability to:

- **Create virtual switches** in Hyper-V for network segmentation
- **Build a star topology** with a central switch connecting multiple VMs
- **Implement a full mesh topology** with NIC teaming for redundancy
- **Configure multiple NICs** per VM for high-availability scenarios
- **Document topology diagrams** showing network connectivity patterns
- **Analyze failure scenarios** in each topology type

## Required Environment

### Hardware/Software

- **Hypervisor**: Hyper-V on Windows Server 2022 (ACIDM01)
- **Guest VMs**:
    - Windows Server 2022 (Member Server) – ACIDM01 (Hyper-V host)
    - Tiny Core Linux VMs (created during lab)
- **Tools**:
    - Hyper-V Manager
    - Virtual Switch Manager
    - Network diagram tool (draw.io, Visio, or hand-drawn)

### Host System

- **OS**: Windows Server 2022 or Windows 11 Pro with Hyper-V enabled
- **RAM**: 16GB minimum (nested virtualization requires host RAM)
- **Storage**: 40GB free (for VM hard disks)
- **CPU**: Virtualization extensions enabled (Intel VT-x / AMD-V)

## Deliverables

### Evidence Files (Required)

1. `01-hyper-v-virtual-switch-manager.png` – Virtual Switch Manager showing all created switches
2. `02-star-topology-switch-creation.png` – StarSwitch configuration details
3. `03-star-vm1-network-adapter-config.png` – VM1 connected to StarSwitch
4. `04-star-vm2-network-adapter-config.png` – VM2 connected to StarSwitch
5. `05-star-vm3-network-adapter-config.png` – VM3 connected to StarSwitch
6. `06-star-topology-diagram.png` – Visual diagram showing star topology
7. `07-fullmesh-switch-ws1-creation.png` – WS1 switch configuration
8. `08-fullmesh-switch-ws2-creation.png` – WS2 switch configuration
9. `09-fullmesh-switch-ws3-creation.png` – WS3 switch configuration
10. `10-workstation1-nic-teaming-config.png` – Workstation1 with 3 NICs enabled for teaming
11. `11-workstation2-nic-teaming-config.png` – Workstation2 with 3 NICs enabled for teaming
12. `12-workstation3-nic-teaming-config.png` – Workstation3 with 3 NICs enabled for teaming
13. `13-full-mesh-topology-diagram.png` – Visual diagram showing full mesh topology

### Documentation (Required)

- **Topology comparison matrix**: Star vs. Full Mesh (advantages/disadvantages)
- **Failure scenario analysis**: What happens if one link/switch fails in each topology?
- **Use case recommendations**: When to deploy each topology type

## Lab Tasks

### Phase 1: Star Topology Implementation

**Estimated Time**: 60 minutes

**Objective**: Create a centralized star topology where all VMs connect through a single virtual switch.

#### Task 1.1: Create Virtual Switch

- [ ] **Step 1**: Connect to ACIDM01 (Hyper-V host)
    
    ```plaintext
    Launch Hyper-V Manager from Taskbar
    Actions pane → Virtual Switch Manager
    ```
    
- [ ] **Step 2**: Create internal switch
    
    ```plaintext
    Select "Internal" virtual switch type
    Click "Create Virtual Switch"
    Name: StarSwitch
    Notes: "Star topology demonstration switch"
    Click OK
    ```
    
- [ ] **Step 3**: Capture evidence
    
    - **Screenshot**: Virtual Switch Manager showing StarSwitch
        - Save as: `01-hyper-v-virtual-switch-manager.png`
    - **Screenshot**: StarSwitch properties
        - Save as: `02-star-topology-switch-creation.png`

#### Task 1.2: Create Virtual Machine 1 (Star Node)

- [ ] **Step 4**: Create new VM
    
    ```plaintext
    Actions → New → Virtual Machine
    
    Name: StarVM1
    Generation: 1
    Memory: 512MB (default)
    Network: StarSwitch
    Virtual Hard Disk: Create new (8GB)
    Installation Options: Skip (no OS needed for topology demo)
    ```
    
- [ ] **Step 5**: Verify network adapter connection
    
    ```plaintext
    Right-click StarVM1 → Settings
    Network Adapter → Virtual switch: StarSwitch
    ```
    
- [ ] **Step 6**: Capture evidence
    
    - **Screenshot**: StarVM1 network adapter configuration
        - Save as: `03-star-vm1-network-adapter-config.png`

#### Task 1.3: Create Virtual Machines 2 & 3

- [ ] **Step 7**: Repeat VM creation process
    
    ```plaintext
    Create StarVM2 with same settings
    Create StarVM3 with same settings
    All VMs connected to StarSwitch
    ```
    
- [ ] **Step 8**: Capture evidence
    
    - **Screenshot**: StarVM2 network config
        - Save as: `04-star-vm2-network-adapter-config.png`
    - **Screenshot**: StarVM3 network config
        - Save as: `05-star-vm3-network-adapter-config.png`

#### Task 1.4: Document Star Topology

- [ ] **Step 9**: Create topology diagram
    
    ```markdown
    ## Star Topology Diagram
    
         [StarVM1]
             |
         [StarVM2]--[StarSwitch]--[StarVM3]
                        |
                    (Central Point)
    
    - All VMs connect to single switch
    - Switch acts as central hub
    - Failure of switch = total network outage
    ```
    
- [ ] **Step 10**: Capture diagram
    
    - **Screenshot/Drawing**: Star topology visual
        - Save as: `06-star-topology-diagram.png`

**Expected Result**: 3 VMs connected to a single virtual switch, forming a star topology.

---

### Phase 2: Full Mesh Topology Implementation

**Estimated Time**: 90 minutes

**Objective**: Create a full mesh network where each VM has direct connections to every other VM using multiple virtual switches and NIC teaming.

#### Task 2.1: Create Three Virtual Switches

- [ ] **Step 1**: Create WS1 switch
    
    ```plaintext
    Hyper-V Manager → Virtual Switch Manager
    Type: Internal
    Name: WS1
    Click OK
    ```
    
- [ ] **Step 2**: Create WS2 switch
    
    ```plaintext
    Type: Internal
    Name: WS2
    Click OK
    ```
    
- [ ] **Step 3**: Create WS3 switch
    
    ```plaintext
    Type: Internal
    Name: WS3
    Click OK
    ```
    
- [ ] **Step 4**: Capture evidence
    
    - **Screenshot**: WS1 configuration
        - Save as: `07-fullmesh-switch-ws1-creation.png`
    - **Screenshot**: WS2 configuration
        - Save as: `08-fullmesh-switch-ws2-creation.png`
    - **Screenshot**: WS3 configuration
        - Save as: `09-fullmesh-switch-ws3-creation.png`

#### Task 2.2: Create Workstation1 with Multiple NICs

- [ ] **Step 5**: Create Workstation1 VM
    
    ```plaintext
    Actions → New → Virtual Machine
    Name: Workstation1
    Generation: 1
    Memory: 512MB
    Network: WS1 (first adapter)
    Create virtual hard disk: 8GB
    ```
    
- [ ] **Step 6**: Add second NIC
    
    ```plaintext
    Right-click Workstation1 → Settings
    Add Hardware → Network Adapter → Add
    Virtual switch: WS2
    Click Apply
    ```
    
- [ ] **Step 7**: Expand Network Adapter WS2
    
    ```plaintext
    Settings → Hardware → Expand "Network Adapter WS2"
    Select "Advanced Features"
    Scroll down to "NIC Teaming"
    Check: "Enable this network adapter to be part of a team in the guest OS"
    Click Apply
    ```
    
- [ ] **Step 8**: Add third NIC
    
    ```plaintext
    Add Hardware → Network Adapter → Add
    Virtual switch: WS3
    Advanced Features → Enable NIC Teaming
    Click Apply
    ```
    
- [ ] **Step 9**: Verify all three NICs
    
    ```plaintext
    Workstation1 should now have:
    - Network Adapter WS1 (NIC teaming enabled)
    - Network Adapter WS2 (NIC teaming enabled)
    - Network Adapter WS3 (NIC teaming enabled)
    ```
    
- [ ] **Step 10**: Capture evidence
    
    - **Screenshot**: Workstation1 showing all 3 NICs with teaming enabled
        - Save as: `10-workstation1-nic-teaming-config.png`

#### Task 2.3: Create Workstation2 and Workstation3

- [ ] **Step 11**: Replicate for Workstation2
    
    ```plaintext
    Create Workstation2 VM
    Add 3 NICs connected to WS1, WS2, WS3
    Enable NIC teaming on all adapters
    ```
    
- [ ] **Step 12**: Capture evidence
    
    - **Screenshot**: Workstation2 NIC configuration
        - Save as: `11-workstation2-nic-teaming-config.png`
- [ ] **Step 13**: Replicate for Workstation3
    
    ```plaintext
    Create Workstation3 VM
    Add 3 NICs connected to WS1, WS2, WS3
    Enable NIC teaming on all adapters
    ```
    
- [ ] **Step 14**: Capture evidence
    
    - **Screenshot**: Workstation3 NIC configuration
        - Save as: `12-workstation3-nic-teaming-config.png`

#### Task 2.4: Document Full Mesh Topology

- [ ] **Step 15**: Create topology diagram
    
    ```markdown
    ## Full Mesh Topology Diagram
    
    [Workstation1]--WS1--[Workstation2]
         |               |
        WS2             WS3
         |               |
    [Workstation3]------+
    
    Each workstation connects to ALL other workstations via:
    - WS1 switch (Workstation1 ↔ Workstation2 ↔ Workstation3)
    - WS2 switch (Workstation1 ↔ Workstation3)
    - WS3 switch (Workstation2 ↔ Workstation3)
    
    Formula: n(n-1)/2 connections
    3 workstations = 3(3-1)/2 = 3 unique paths
    ```
    
- [ ] **Step 16**: Capture diagram
    
    - **Screenshot/Drawing**: Full mesh topology visual
        - Save as: `13-full-mesh-topology-diagram.png`

**Expected Result**: 3 VMs with full connectivity to each other via 3 virtual switches, forming a full mesh topology.

---

## Observations & Failure Modes

### Common Issues

#### Issue 1: "Nested Virtualization Not Supported" Error

**Cause**: Host CPU doesn't support virtualization extensions or Hyper-V not enabled  
**Solution**:

```powershell
# Enable Hyper-V on Windows 11 Pro
Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V-All

# Enable nested virtualization (run on host, VM powered off)
Set-VMProcessor -VMName ACIDM01 -ExposeVirtualizationExtensions $true
```

#### Issue 2: NIC Teaming Option Grayed Out

**Cause**: Virtual switch type is "External" instead of "Internal"  
**Solution**:

- Delete existing switch
- Recreate as "Internal" type
- Internal switches allow NIC teaming within guest OS

#### Issue 3: VMs Cannot Communicate in Full Mesh

**Cause**: Guest OS networking not configured (no IP addresses assigned)  
**Solution**:

```bash
# Inside Tiny Core Linux VM (if booted)
sudo ifconfig eth0 192.168.1.1 netmask 255.255.255.0
sudo ifconfig eth1 192.168.2.1 netmask 255.255.255.0
sudo ifconfig eth2 192.168.3.1 netmask 255.255.255.0

# Test connectivity
ping 192.168.1.2  # Reach other VM
```

**Note**: For topology demonstration purposes, assigning IPs is optional. Focus on physical connectivity.

#### Issue 4: Hyper-V Manager Shows No VMs

**Cause**: Wrong Hyper-V host selected in manager  
**Solution**:

```plaintext
Hyper-V Manager → Connect to Server
Enter: localhost (or ACIDM01 hostname)
Verify VMs appear in middle pane
```

---

## Analysis & Reflection

### Topology Comparison Matrix

|Feature|Star Topology|Full Mesh Topology|
|---|---|---|
|**Complexity**|Low – Single switch|High – Multiple switches|
|**Cost**|Low – Minimal hardware|High – More NICs/switches|
|**Failure Impact**|High – Switch failure = total outage|Low – Multiple redundant paths|
|**Bandwidth**|Shared across all nodes|Dedicated per connection|
|**Scalability**|Easy to add nodes|Complex (n² connections)|
|**Troubleshooting**|Simple (one point)|Complex (many paths)|
|**Best Use Case**|Branch offices, VDI farms|HA clusters, SANs, core switches|

### Failure Scenario Analysis

**Star Topology Failure**:

```
Scenario: StarSwitch fails
Impact: ALL VMs lose connectivity
Recovery: Replace/restore switch
Mitigation: Redundant switch with failover
```

**Full Mesh Topology Failure**:

```
Scenario: WS1 switch fails
Impact: Only WS1 connections lost, WS2 and WS3 paths remain
Recovery: Automatic failover via NIC teaming
Mitigation: Already redundant (design feature)
```

### Real-World Applications

**Star Topology – Corporate Office**:

- 50-user office with single access switch
- Cost-effective for non-critical workloads
- Centralized management (VLAN tagging on single switch)
- Risk: Switch failure disrupts all users

**Full Mesh Topology – Database Cluster**:

- 3-node SQL Server Always On cluster
- Each node needs high-bandwidth replication
- Any single link failure doesn't affect cluster quorum
- Example: Microsoft SQL Server AlwaysOn Availability Groups

**Hybrid Approach – Enterprise Network**:

```
Star at Access Layer:
  [User PCs] → [Access Switch]
       ↓
Full Mesh at Core Layer:
  [Core Switch 1] ←→ [Core Switch 2] ←→ [Core Switch 3]
```

### Security Considerations

**Star Topology**:

- **Pro**: Single point to implement ACLs, VLANs, port security
- **Con**: Switch compromise affects entire network

**Full Mesh Topology**:

- **Pro**: Segmentation across multiple switches limits blast radius
- **Con**: More attack surface (multiple switches to secure)

### Time Tracking

|Phase|Estimated|Actual|Notes|
|---|---|---|---|
|Phase 1: Star Topology|60 min|___ min||
|Phase 2: Full Mesh|90 min|___ min||
|**Total**|**150 min**|**___ min**||

---

## Exam Relevance Notes

### Exam Scenario Mapping

**N10-009 Performance-Based Questions (PBQs)**:

- **Diagram a network topology based on requirements**
    - Given: "Design a network for 100 users, single building, cost-effective"
    - Answer: Star topology with central switch
- **Identify topology type from diagram**
    - Given: Network diagram showing all nodes interconnected
    - Answer: Full mesh topology

**Multiple Choice Question Patterns**:

- "Which topology provides the most redundancy?"
    - Answer: Mesh (full or partial)
- "A network has a central switch connecting all devices. This describes which topology?"
    - Answer: Star
- "What is the formula for number of connections in a full mesh network?"
    - Answer: n(n-1)/2 where n = number of nodes

### Key Exam Concepts Reinforced

**Topology Characteristics**:

- **Bus**: Single cable, easy to tap, obsolete
- **Ring**: Token passing, dual-ring for redundancy (FDDI, Token Ring)
- **Star**: Central device (switch/hub), most common in LANs
- **Mesh**: Full (all interconnected) vs. Partial (some interconnected)
- **Hybrid**: Combination (star-mesh in enterprise core)

**NIC Teaming / Link Aggregation**:

- Combines multiple NICs for:
    - Increased bandwidth (load balancing)
    - Redundancy (failover)
- Protocols: LACP (802.3ad), Static teaming
- Exam terms: "Bonding", "Aggregation", "Teaming"

---

## Submission Checklist

Before marking this lab complete, verify:

- [ ] All 13 evidence files collected and named correctly
- [ ] Screenshots show Hyper-V Manager with VM configurations visible
- [ ] Both topology diagrams created (hand-drawn or digital)
- [ ] Topology comparison matrix completed
- [ ] Failure scenario analysis documented
- [ ] Time tracking completed
- [ ] Evidence files moved to `/mnt/user-data/outputs`

**Final Evidence Count**: 13 files  
**Documentation Pages**: 1 (this lab markdown)  
**Ready for GitHub**: Yes / No

---

## Next Steps

After completing this lab, you should:

1. **Review**: Study hybrid topologies (how star + mesh combine in enterprise)
2. **Practice**: Build a three-tier network (access/distribution/core) using Hyper-V
3. **Advance**: Proceed to Lab 03 (Switching Concepts) to implement VLANs on these topologies
4. **Optional Challenge**: Add pfSense router to route between star and mesh topologies

**Additional Resources**:

- [Microsoft Hyper-V Documentation](https://docs.microsoft.com/en-us/virtualization/hyper-v-on-windows/)
- CompTIA Network+ N10-009 Official Study Guide, Chapter 1 (Topologies)
- [Cisco: Network Topologies Overview](https://learningnetwork.cisco.com/)