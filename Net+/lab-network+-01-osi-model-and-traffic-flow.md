---
title: OSI Model Layer Analysis and Traffic Flow
difficulty: Foundation
Cert:
  - Network+
  - NT10-009
Exam-Objectives:
  - Net+ 1.1
created: 2026-02-13
completed:
time-hours: 2
complete: false
GitHub-ready: false
evidence-files: 10
notes:
tags:
  - network-plus-nt0-009
  - osi-model
  - wireshark
  - network-protocols
  - layer-analysis
---
---
## Objective Mapping

**1.1 Explain concepts related to the Open Systems Interconnection (OSI) reference model**

- Layer 1 – Physical
- Layer 2 – Data link
- Layer 3 – Network
- Layer 4 – Transport
- Layer 5 – Session
- Layer 6 – Presentation
- Layer 7 – Application

## Scenario

You're a junior network analyst supporting a mixed Windows/Linux enterprise environment. Your team lead has assigned you to document how network traffic flows through the OSI model during routine operations. This foundational exercise will prepare you for advanced troubleshooting scenarios where you must identify which OSI layer is failing during network incidents.

Your organization uses Windows Server 2022 domain controllers, Windows 11 workstations, and Alma Linux 9.3 systems for development environments, all managed through a pfSense virtual router. Today's task focuses on **building a mental map** of the OSI model by observing real network components, interface statistics, protocol behavior, and application-layer services.

This lab mirrors a common SOC onboarding exercise: "Show me you understand what's happening at each layer when a user accesses a file share or sends DNS queries."

## Learning Objectives

By completing this lab, you will demonstrate ability to:

- **Map physical network components** (NICs, adapters) to Layer 1 operations
- **Analyze Layer 2 data link operations** via interface statistics and ARP tables
- **Examine Layer 3 network routing** using route print and routing tables
- **Observe Layer 4 transport mechanisms** with TCP handshakes in Wireshark
- **Identify Layer 5 session establishment** through TCP stream following
- **Demonstrate Layer 6 presentation functions** via file compression and encryption
- **Interact with Layer 7 application protocols** using nslookup for DNS resolution

## Required Environment

### Hardware/Software

- **Hypervisor**: VMware Workstation Pro or Hyper-V
- **Guest VMs**:
    - Windows Server 2022 (Domain Controller) – ACIDC01
    - Windows 11 Pro (Domain Member) – ACIWIN11
    - pfSense v2.7.2 (Virtual Router) – ACIPFSENSE
- **Tools**:
    - Device Manager (Layer 1 inspection)
    - Windows Command Prompt (ipconfig, arp, route, nslookup)
    - Wireshark (packet capture and protocol analysis)
    - Windows built-in compression/encryption

### Host System

- **OS**: Windows 11 Pro or Windows Server 2022
- **RAM**: 16GB minimum (8GB allocated to VMs)
- **Network**: Virtualized internal network (no external internet required)

## Deliverables

### Evidence Files (Required)

1. `01-nic-properties-general-tab.png` – NIC status showing "Device working properly"
2. `02-nic-events-tab.png` – Installation/configuration events from Device Manager
3. `03-ipconfig-all-output.txt` – Full network adapter configuration
4. `04-arp-table-output.txt` – Current ARP cache showing MAC-to-IP mappings
5. `05-route-print-output.txt` – IPv4 routing table
6. `06-wireshark-tcp-handshake.png` – SYN, SYN-ACK, ACK sequence captured
7. `07-wireshark-tcp-stream-follow.png` – HTTP stream or DNS query/response
8. `08-compressed-file-properties.png` – File size before/after compression
9. `09-encrypted-file-properties.png` – Encryption status confirmation
10. `10-nslookup-query-output.txt` – DNS query for internal domain controller

### Documentation (Required)

- **Lab notes**: Observations about what each OSI layer handles
- **Failure modes encountered**: Document at least one troubleshooting step taken
- **Time tracking**: Actual time per phase vs. estimated

## Lab Tasks

### Phase 1: Physical Layer (Layer 1) – Hardware Inspection

**Estimated Time**: 15 minutes

**Objective**: Examine the NIC as a physical-layer device responsible for electrical signal transmission.

- [ ] **Task 1.1**: Connect to ACIWIN11 VM
    
    - Power on ACIWIN11
    - Log in with domain credentials
- [ ] **Task 1.2**: Access Device Manager
    
    ```plaintext
    Right-click Start → Device Manager
    Expand "Network adapters"
    Right-click "Microsoft Hyper-V Network Adapter" → Properties
    ```
    
- [ ] **Task 1.3**: Capture Layer 1 evidence
    
    - **Screenshot**: General tab showing device status
        - Save as: `01-nic-properties-general-tab.png`
    - **Screenshot**: Events tab showing installation/activation events
        - Save as: `02-nic-events-tab.png`
- [ ] **Task 1.4**: Document Layer 1 observations
    
    ```markdown
    ## Layer 1 Notes
    - Physical medium: [Virtual NIC type]
    - Status: [Working properly / Error]
    - Key events: [Device installed, configured, started]
    ```
    

**Expected Result**: You've verified the NIC operates at Layer 1 by transmitting binary signals over virtual hardware.

---

### Phase 2: Data Link Layer (Layer 2) – MAC Addressing & Switching

**Estimated Time**: 20 minutes

**Objective**: Examine Layer 2 operations including MAC addresses, ARP resolution, and frame handling.

- [ ] **Task 2.1**: Run ipconfig /all
    
    ```cmd
    ipconfig /all > C:\Evidence\03-ipconfig-all-output.txt
    ```
    
    - Identify:
        - Physical Address (MAC)
        - IPv4 Address
        - Default Gateway
- [ ] **Task 2.2**: Examine ARP table
    
    ```cmd
    arp -a > C:\Evidence\04-arp-table-output.txt
    ```
    
    - Review mappings of IP addresses to MAC addresses
    - Note dynamic vs. static entries
- [ ] **Task 2.3**: Ping default gateway to populate ARP
    
    ```cmd
    ping [gateway-ip]
    arp -a
    ```
    
    - Observe new ARP entry after ping

**Evidence Required**:

- `03-ipconfig-all-output.txt`
- `04-arp-table-output.txt`

**Expected Result**: You understand Layer 2 uses MAC addresses to deliver frames within the local network segment.

---

### Phase 3: Network Layer (Layer 3) – IP Routing

**Estimated Time**: 15 minutes

**Objective**: Analyze the routing table to understand how Layer 3 determines packet forwarding paths.

- [ ] **Task 3.1**: Display IPv4 routing table
    
    ```cmd
    route print > C:\Evidence\05-route-print-output.txt
    ```
    
- [ ] **Task 3.2**: Identify key route types
    
    - **Default route** (0.0.0.0/0): Gateway to external networks
    - **Local subnet routes**: Direct delivery
    - **Host routes**: Specific destination entries
- [ ] **Task 3.3**: Test connectivity using ping
    
    ```cmd
    ping 192.168.0.1
    ping acidc01.aciplab.com
    ```
    

**Evidence Required**:

- `05-route-print-output.txt`

**Expected Result**: You've demonstrated Layer 3 routing decisions based on IP addresses.

---

### Phase 4: Transport Layer (Layer 4) – TCP Connection Establishment

**Estimated Time**: 25 minutes

**Objective**: Use Wireshark to observe the TCP three-way handshake.

- [ ] **Task 4.1**: Launch Wireshark on ACIWIN11
    
    - Select network adapter
    - Start packet capture
- [ ] **Task 4.2**: Generate TCP traffic
    
    ```cmd
    # Open browser to http://acidc01
    # OR initiate RDP connection
    mstsc /v:acidc01
    ```
    
- [ ] **Task 4.3**: Stop capture and filter
    
    ```wireshark-filter
    tcp.flags.syn==1
    ```
    
    - Locate SYN, SYN-ACK, ACK sequence
    - Note source/destination ports
- [ ] **Task 4.4**: Capture evidence
    
    - **Screenshot**: Three-way handshake packets
        - Save as: `06-wireshark-tcp-handshake.png`

**Evidence Required**:

- `06-wireshark-tcp-handshake.png`

**Expected Result**: You've identified Layer 4 responsible for reliable end-to-end connections.

---

### Phase 5: Session Layer (Layer 5) – Stream Following

**Estimated Time**: 10 minutes

**Objective**: Follow a TCP stream to see session establishment between client and server.

- [ ] **Task 5.1**: In Wireshark, right-click a TCP packet
    
    ```plaintext
    Follow → TCP Stream
    ```
    
- [ ] **Task 5.2**: Observe session data
    
    - HTTP headers (if web traffic)
    - DNS queries/responses (if DNS traffic)
    - SMB session negotiation (if file share)
- [ ] **Task 5.3**: Capture evidence
    
    - **Screenshot**: TCP stream window showing session data
        - Save as: `07-wireshark-tcp-stream-follow.png`

**Evidence Required**:

- `07-wireshark-tcp-stream-follow.png`

**Expected Result**: You've demonstrated Layer 5 manages sessions between applications.

---

### Phase 6: Presentation Layer (Layer 6) – Data Transformation

**Estimated Time**: 20 minutes

**Objective**: Demonstrate compression and encryption as Layer 6 functions.

#### Compression Test

- [ ] **Task 6.1**: Create test file
    
    ```cmd
    echo This is a test file with repetitive content > C:\Evidence\testfile.txt
    # Copy/paste content multiple times to increase size
    ```
    
- [ ] **Task 6.2**: Enable NTFS compression
    
    ```plaintext
    Right-click testfile.txt → Properties
    Advanced → Check "Compress contents to save disk space"
    Apply → OK
    ```
    
- [ ] **Task 6.3**: Capture evidence
    
    - **Screenshot**: Properties showing compressed size vs. actual size
        - Save as: `08-compressed-file-properties.png`

#### Encryption Test

- [ ] **Task 6.4**: Enable EFS encryption
    
    ```plaintext
    Right-click testfile.txt → Properties
    Advanced → Check "Encrypt contents to secure data"
    Apply → OK
    ```
    
- [ ] **Task 6.5**: Capture evidence
    
    - **Screenshot**: Properties showing encryption status
        - Save as: `09-encrypted-file-properties.png`

**Evidence Required**:

- `08-compressed-file-properties.png`
- `09-encrypted-file-properties.png`

**Expected Result**: You've shown Layer 6 handles data format translation, compression, and encryption.

---

### Phase 7: Application Layer (Layer 7) – DNS Resolution

**Estimated Time**: 10 minutes

**Objective**: Use nslookup to interact with DNS as an application-layer service.

- [ ] **Task 7.1**: Query internal DNS server
    
    ```cmd
    nslookup acidc01.aciplab.com > C:\Evidence\10-nslookup-query-output.txt
    ```
    
- [ ] **Task 7.2**: Perform reverse lookup
    
    ```cmd
    nslookup 192.168.0.10 >> C:\Evidence\10-nslookup-query-output.txt
    ```
    
- [ ] **Task 7.3**: Identify DNS server
    
    - Note which server responded
    - Verify authoritative vs. non-authoritative answer

**Evidence Required**:

- `10-nslookup-query-output.txt`

**Expected Result**: You've demonstrated Layer 7 application protocols (DNS) provide services to end users.

---

## Observations & Failure Modes

### Common Issues

#### Issue 1: Wireshark Shows No Packets

**Cause**: Wrong network adapter selected or insufficient permissions  
**Solution**:

- Run Wireshark as Administrator
- Select "Microsoft Hyper-V Network Adapter" (not loopback)
- Generate traffic while capture is active

#### Issue 2: ARP Table Empty After arp -a

**Cause**: No recent network communication  
**Solution**:

```cmd
ping [default-gateway]
arp -a
```

#### Issue 3: Encryption Option Grayed Out

**Cause**: NTFS drive required for EFS  
**Solution**:

- Verify C:\ is NTFS (not FAT32)
- Check user has encryption permissions
- Run `cipher /e /s:C:\Evidence` if GUI fails

#### Issue 4: TCP Handshake Not Visible in Wireshark

**Cause**: Existing connection already established  
**Solution**:

- Close browser/RDP completely
- Clear Wireshark capture
- Start new capture BEFORE initiating connection

---

## Analysis & Reflection

### Security Considerations

**Layer 1 (Physical)**:

- **Risk**: Physical access to NICs enables MAC spoofing, cable tapping
- **Mitigation**: Secure server rooms, use locked network cabinets

**Layer 2 (Data Link)**:

- **Risk**: ARP spoofing/poisoning attacks redirect traffic to attacker
- **Mitigation**: Static ARP entries for critical infrastructure, DHCP snooping

**Layer 3 (Network)**:

- **Risk**: IP spoofing, routing table manipulation
- **Mitigation**: Ingress/egress filtering, route authentication (BGP/OSPF)

**Layer 4 (Transport)**:

- **Risk**: SYN flood attacks exhaust TCP connection table
- **Mitigation**: SYN cookies, rate limiting, firewall rules

**Layer 5 (Session)**:

- **Risk**: Session hijacking via stolen tokens/cookies
- **Mitigation**: Session timeouts, token rotation, encrypted sessions

**Layer 6 (Presentation)**:

- **Risk**: Weak encryption algorithms (DES, RC4) enable decryption
- **Mitigation**: Use AES-256, TLS 1.3, disable legacy ciphers

**Layer 7 (Application)**:

- **Risk**: DNS cache poisoning returns malicious IP addresses
- **Mitigation**: DNSSEC validation, secure DNS resolvers

### Real-World Applications

**Incident Response Workflow**:

1. User reports "network is slow"
2. Check Layer 1: Cable plugged in? Link light active?
3. Check Layer 2: ARP table correct? Switch port operational?
4. Check Layer 3: Can ping default gateway? Routing table correct?
5. Check Layer 4-7: TCP retransmissions? Application errors?

**SOC Analyst Use Case**:

- **Wireshark analysis**: Identify malicious DNS queries (Layer 7)
- **ARP monitoring**: Detect ARP spoofing (Layer 2)
- **Route auditing**: Verify no unauthorized static routes (Layer 3)

### Time Tracking

|Phase|Estimated|Actual|Notes|
|---|---|---|---|
|Phase 1: Layer 1|15 min|___ min||
|Phase 2: Layer 2|20 min|___ min||
|Phase 3: Layer 3|15 min|___ min||
|Phase 4: Layer 4|25 min|___ min||
|Phase 5: Layer 5|10 min|___ min||
|Phase 6: Layer 6|20 min|___ min||
|Phase 7: Layer 7|10 min|___ min||
|**Total**|**115 min**|**___ min**||

---

## Exam Relevance Notes

### Exam Scenario Mapping

**N10-009 Performance-Based Questions (PBQs)**:

- **Given a network diagram, identify which OSI layer a device operates at**
    - Router → Layer 3
    - Switch → Layer 2
    - Hub → Layer 1
- **Troubleshoot connectivity using OSI model top-down or bottom-up**
    - Bottom-up: Check physical cable → Link lights → IP config → Ping gateway
    - Top-down: Check application → DNS → Routing → ARP → Physical

**Multiple Choice Question Patterns**:

- "Which layer is responsible for [MAC addressing / routing / encryption]?"
- "A user cannot access a file share. Which tool checks Layer 3 connectivity?"
    - Answer: `ping` (ICMP operates at Layer 3)

### Key Exam Concepts Reinforced

- **Encapsulation/De-encapsulation**: Each layer adds headers
    
    - Layer 2: Frame (MAC header)
    - Layer 3: Packet (IP header)
    - Layer 4: Segment (TCP/UDP header)
- **Protocol Data Units (PDUs)**:
    
    - Layer 1: Bits
    - Layer 2: Frames
    - Layer 3: Packets
    - Layer 4: Segments/Datagrams
    - Layer 5-7: Data
- **Troubleshooting Order**:
    
    - Physical first (cables, lights)
    - Then logical (IP config, routing)
    - Finally application (DNS, web services)

---

## Submission Checklist

Before marking this lab complete, verify:

- [ ] All 10 evidence files collected and named correctly
- [ ] Screenshots show relevant information (not blank windows)
- [ ] Text outputs (`ipconfig`, `arp`, `route`, `nslookup`) saved as .txt files
- [ ] Lab notes document at least one observation per OSI layer
- [ ] At least one failure mode documented with solution
- [ ] Time tracking completed
- [ ] Evidence files moved to `/mnt/user-data/outputs` (if using Claude's Linux environment)

**Final Evidence Count**: 10 files  
**Documentation Pages**: 1 (this lab markdown)  
**Ready for GitHub**: Yes / No

---

## Next Steps

After completing this lab, you should:

1. **Review**: Re-read OSI model definitions in Network+ study guide
2. **Practice**: Build a quick reference card mapping protocols to layers
3. **Advance**: Proceed to Lab 02 (Network Topologies) to apply OSI knowledge in network design
4. **Optional Challenge**: Repeat Wireshark capture with FTP traffic to see all 7 layers in action

**Additional Resources**:

- [Wireshark User Guide](https://www.wireshark.org/docs/wsug_html_chunked/)
- CompTIA Network+ N10-009 Official Study Guide, Chapter 1
- [Cisco: OSI Model Explained](https://learningnetwork.cisco.com/)