---
title: Network Troubleshooting and Wireless Configuration
difficulty: Intermediate
Cert:
  - Network+
  - NT10-009
Exam-Objectives:
created: 2026-02-13
completed:
time-hours: 4
complete: false
GitHub-ready: false
evidence-files: 13
notes:
tags:
  - troubleshooting
  - wireless
  - wireshark
  - tcpdump
  - nmap
  - 802-11-standards
  - wifi-security
  - packet-analysis
---
---
## Objective Mapping

**5.5 Given a scenario, use appropriate network troubleshooting tools**

- Ping, traceroute/tracert
- Nslookup/dig
- nmap
- tcpdump
- Wireshark

**2.4 Given a scenario, install and configure the appropriate wireless standards and technologies**

- 802.11 standards (a/b/g/n/ac/ax)
- Frequencies and range
- Encryption standards (WEP/WPA/WPA2/WPA3)
- SSID configuration

**5.2 Given a scenario, troubleshoot common network issues**

- Connectivity issues
- DNS issues
- Wireless issues

## Scenario

You're the lead network engineer at a healthcare clinic that just upgraded to WiFi 6 (802.11ax) for wireless patient monitoring devices. Since the upgrade, users report:

1. **"Internet is slow"** → Unclear which layer is failing (connectivity, DNS, routing?)
2. **"WiFi keeps dropping"** → Signal strength? Interference? Authentication failures?
3. **"Security scan detected rogue devices"** → Unauthorized APs on the network?

Your manager needs a **systematic troubleshooting report** using industry-standard tools to:

- **Diagnose connectivity** from Layer 1-7 using ping, traceroute, nslookup
- **Identify network threats** using nmap and packet capture
- **Validate wireless security** by comparing WEP/WPA/WPA2/WPA3 configurations

This lab mirrors real SOC workflows where you **use the right tool for the right layer** and **document findings for stakeholders**.

## Learning Objectives

By completing this lab, you will demonstrate ability to:

- **Test Layer 3 connectivity** with ping (ICMP echo request/reply)
- **Trace network paths** with traceroute/tracert to identify routing issues
- **Resolve DNS queries** using nslookup and dig for authoritative lookups
- **Scan networks** with nmap for host discovery and open ports
- **Capture packets** with tcpdump for offline analysis
- **Analyze traffic** with Wireshark to decrypt protocols and identify attacks
- **Compare wireless encryption** standards (WEP/WPA/WPA2/WPA3)
- **Configure SSIDs** with proper security settings

## Required Environment

### Hardware/Software

- **VMs**:
    - ACIALMA (Alma Linux 9.3) - tcpdump, nmap, dig
    - ACIDC01 (Windows Server 2022 DC) - DNS server
    - ACIWIN11 (Windows 11 Pro) - Wireshark, ping, tracert, nslookup
- **Tools**:
    - ping, tracert (Windows) / traceroute (Linux)
    - nslookup (Windows) / dig (Linux)
    - nmap, tcpdump, Wireshark
- **Wireless**: Virtual or physical WiFi adapter (optional - can use documentation)

### Host System

- **OS**: Windows 11 Pro or Windows Server 2022
- **RAM**: 12GB minimum
- **Network**: Internal virtual network + internet access for external trace

## Deliverables

### Evidence Files (Required)

1. `01-ping-google-dns-success.txt` - Verify internet connectivity
2. `02-ping-internal-dc-success.txt` - Verify local network connectivity
3. `03-tracert-google-com-path.txt` - Show route to external destination
4. `04-traceroute-internal-dc-path.txt` - Show hops to domain controller
5. `05-nslookup-acidc01-forward-lookup.txt` - DNS A record query
6. `06-nslookup-reverse-lookup-ptr.txt` - DNS PTR record query
7. `07-dig-authoritative-dns-query.txt` - Authoritative nameserver response
8. `08-nmap-host-discovery-sweep.txt` - Live hosts in subnet
9. `09-nmap-port-scan-top-1000.txt` - Open ports on target host
10. `10-tcpdump-packet-capture.pcap` - Captured network traffic
11. `11-wireshark-tcp-syn-scan-analysis.png` - Nmap scan visible in Wireshark
12. `12-wireless-encryption-comparison-table.md` - WEP/WPA/WPA2/WPA3 comparison
13. `13-wifi-ssid-security-config.png` - Recommended WiFi settings

### Documentation (Required)

- **Troubleshooting decision tree**: Flowchart for diagnosing network issues
- **Tool selection matrix**: Which tool to use for each problem type
- **Wireless security recommendations**: Standards to deploy vs. deprecate

## Lab Tasks

### Phase 1: Connectivity Troubleshooting (60 minutes)

**Objective**: Use ping and traceroute to identify where network communication fails.

#### Task 1.1: Test Internet Connectivity (Layer 3)

- [ ] **Step 1**: On ACIWIN11, open Command Prompt
    
- [ ] **Step 2**: Ping external DNS server
    
    ```cmd
    ping 8.8.8.8 > C:\Evidence\01-ping-google-dns-success.txt
    type C:\Evidence\01-ping-google-dns-success.txt
    ```
    
    **Expected Output**:
    
    ```
    Reply from 8.8.8.8: bytes=32 time=15ms TTL=118
    Reply from 8.8.8.8: bytes=32 time=14ms TTL=118
    
    Ping statistics for 8.8.8.8:
        Packets: Sent = 4, Received = 4, Lost = 0 (0% loss)
    ```
    
    **Conclusion**: Layer 3 connectivity to internet = WORKING
    
- [ ] **Evidence**: `01-ping-google-dns-success.txt`
    

#### Task 1.2: Test Local Network Connectivity

- [ ] **Step 3**: Ping internal domain controller
    
    ```cmd
    ping acidc01.aciplab.com > C:\Evidence\02-ping-internal-dc-success.txt
    type C:\Evidence\02-ping-internal-dc-success.txt
    ```
    
    **Expected Output**:
    
    ```
    Reply from 192.168.0.10: bytes=32 time<1ms TTL=128
    ```
    
    **Conclusion**: Local network connectivity = WORKING
    
- [ ] **Evidence**: `02-ping-internal-dc-success.txt`
    

**Troubleshooting Logic**:

```
Internet ping fails? → Check gateway, DNS, routing
Local ping fails? → Check cable, switch, IP config
Both fail? → Check Layer 1/2 (NIC, switch port)
```

#### Task 1.3: Trace Route to External Destination

- [ ] **Step 4**: Run traceroute to Google
    
    ```cmd
    tracert www.google.com > C:\Evidence\03-tracert-google-com-path.txt
    type C:\Evidence\03-tracert-google-com-path.txt
    ```
    
    **Expected Output** (varies by ISP):
    
    ```
    Tracing route to www.google.com [142.250.80.100]
    
      1    <1 ms    <1 ms    <1 ms  192.168.0.1     [Internal gateway]
      2     5 ms     4 ms     5 ms  10.0.0.1        [ISP router]
      3    12 ms    11 ms    10 ms  72.14.219.100   [Google edge]
      4    15 ms    14 ms    15 ms  142.250.80.100  [www.google.com]
    ```
    
    **Analysis**: Each hop shows latency. High latency (>100ms) or timeouts indicate problem area.
    
- [ ] **Evidence**: `03-tracert-google-com-path.txt`
    

#### Task 1.4: Trace Route to Internal Host (Linux)

- [ ] **Step 5**: On ACIALMA terminal
    
    ```bash
    traceroute acidc01.aciplab.com > ~/04-traceroute-internal-dc-path.txt 2>&1
    cat ~/04-traceroute-internal-dc-path.txt
    ```
    
    **Expected Output**:
    
    ```
    traceroute to acidc01.aciplab.com (192.168.0.10), 30 hops max
      1  192.168.0.10 (192.168.0.10)  0.351 ms  0.289 ms  0.276 ms
    ```
    
    **Conclusion**: Direct connection (1 hop = same subnet)
    
- [ ] **Evidence**: `04-traceroute-internal-dc-path.txt`
    

---

### Phase 2: DNS Troubleshooting (30 minutes)

**Objective**: Use nslookup and dig to verify DNS resolution and identify misconfigurations.

#### Task 2.1: Forward DNS Lookup (A Record)

- [ ] **Step 1**: On ACIWIN11, query A record
    
    ```cmd
    nslookup acidc01.aciplab.com > C:\Evidence\05-nslookup-acidc01-forward-lookup.txt
    type C:\Evidence\05-nslookup-acidc01-forward-lookup.txt
    ```
    
    **Expected Output**:
    
    ```
    Server:  acidc01.aciplab.com
    Address:  192.168.0.10
    
    Name:    acidc01.aciplab.com
    Address:  192.168.0.10
    ```
    
- [ ] **Evidence**: `05-nslookup-acidc01-forward-lookup.txt`
    

#### Task 2.2: Reverse DNS Lookup (PTR Record)

- [ ] **Step 2**: Query PTR record
    
    ```cmd
    nslookup 192.168.0.10 > C:\Evidence\06-nslookup-reverse-lookup-ptr.txt
    type C:\Evidence\06-nslookup-reverse-lookup-ptr.txt
    ```
    
    **Expected Output**:
    
    ```
    Server:  acidc01.aciplab.com
    Address:  192.168.0.10
    
    Name:    acidc01.aciplab.com
    Address:  192.168.0.10
    ```
    
- [ ] **Evidence**: `06-nslookup-reverse-lookup-ptr.txt`
    

#### Task 2.3: Authoritative DNS Query with dig (Linux)

- [ ] **Step 3**: On ACIALMA, perform dig query
    
    ```bash
    dig acidc01.aciplab.com @192.168.0.10 +short > ~/07-dig-authoritative-dns-query.txt
    dig acidc01.aciplab.com @192.168.0.10 >> ~/07-dig-authoritative-dns-query.txt
    cat ~/07-dig-authoritative-dns-query.txt
    ```
    
    **Expected Output**:
    
    ```
    192.168.0.10
    
    ;; ANSWER SECTION:
    acidc01.aciplab.com.    3600    IN      A       192.168.0.10
    
    ;; SERVER: 192.168.0.10#53(192.168.0.10)
    ```
    
- [ ] **Evidence**: `07-dig-authoritative-dns-query.txt`
    

**Troubleshooting DNS Issues**:

```
nslookup fails? → Check DNS server in ipconfig
Wrong IP returned? → Clear DNS cache: ipconfig /flushdns
Timeout? → Verify DNS port 53 open in firewall
```

---

### Phase 3: Network Scanning and Threat Detection (60 minutes)

**Objective**: Use nmap to discover hosts and identify open ports (authorized security scanning).

#### Task 3.1: Host Discovery Scan

- [ ] **Step 1**: On ACIALMA, scan subnet for live hosts
    
    ```bash
    sudo nmap -sn 192.168.0.0/24 > ~/08-nmap-host-discovery-sweep.txt 2>&1
    cat ~/08-nmap-host-discovery-sweep.txt
    ```
    
    **Expected Output**:
    
    ```
    Nmap scan report for 192.168.0.1
    Host is up (0.00012s latency).
    
    Nmap scan report for acidc01.aciplab.com (192.168.0.10)
    Host is up (0.00025s latency).
    
    Nmap scan report for aciwin11.aciplab.com (192.168.0.11)
    Host is up (0.00031s latency).
    
    Nmap done: 256 IP addresses (3 hosts up) scanned in 2.45 seconds
    ```
    
- [ ] **Evidence**: `08-nmap-host-discovery-sweep.txt`
    

**Security Note**: `-sn` = ping scan only (no port scan). Use this first to avoid triggering IDS alerts.

#### Task 3.2: Port Scan on Target Host

- [ ] **Step 2**: Scan domain controller ports
    
    ```bash
    sudo nmap -sT -p- 192.168.0.10 > ~/09-nmap-port-scan-top-1000.txt 2>&1
    cat ~/09-nmap-port-scan-top-1000.txt
    ```
    
    **Expected Output** (Windows DC):
    
    ```
    PORT      STATE SERVICE
    53/tcp    open  domain
    88/tcp    open  kerberos-sec
    135/tcp   open  msrpc
    139/tcp   open  netbios-ssn
    389/tcp   open  ldap
    445/tcp   open  microsoft-ds
    3389/tcp  open  ms-wbt-server
    ```
    
- [ ] **Evidence**: `09-nmap-port-scan-top-1000.txt`
    

**Detection Use Case**:

- Expected ports = domain controller services
- Unexpected ports (e.g., 21-FTP, 23-Telnet) = misconfiguration or compromise

---

### Phase 4: Packet Capture and Analysis (30 minutes)

**Objective**: Capture network traffic with tcpdump and analyze with Wireshark.

#### Task 4.1: Capture Packets with tcpdump

- [ ] **Step 1**: On ACIALMA, start packet capture
    
    ```bash
    # Capture 50 packets on eth0 interface
    sudo tcpdump -i eth0 -c 50 -w ~/myCapture.pcap
    
    # In another terminal, generate traffic
    ping -c 10 192.168.0.10
    ```
    
- [ ] **Step 2**: Read capture file
    
    ```bash
    tcpdump -r ~/myCapture.pcap > ~/10-tcpdump-packet-capture.txt 2>&1
    cat ~/10-tcpdump-packet-capture.txt
    ```
    
- [ ] **Step 3**: Copy PCAP to Windows for Wireshark
    
    ```bash
    # Option 1: Use shared folder
    cp ~/myCapture.pcap /mnt/shared/
    
    # Option 2: Upload to file share (as done in original lab)
    # Then download on ACIWIN11
    ```
    
- [ ] **Evidence**: `10-tcpdump-packet-capture.pcap`
    

#### Task 4.2: Analyze with Wireshark

- [ ] **Step 4**: On ACIWIN11, open Wireshark
    
    ```plaintext
    File → Open → Select myCapture.pcap
    ```
    
- [ ] **Step 5**: Filter for TCP SYN packets (nmap scan evidence)
    
    ```wireshark-filter
    tcp.flags.syn==1 && tcp.flags.ack==0
    ```
    
- [ ] **Step 6**: Observe port scan pattern
    
    ```plaintext
    Source IP: 192.168.0.X (ACIALMA)
    Destination IP: 192.168.0.10 (ACIDC01)
    Ports: Sequential (1, 2, 3, 4, 5...)
    
    This pattern = nmap SYN scan
    ```
    
- [ ] **Evidence**:
    
    - **Screenshot**: Wireshark showing TCP SYN packets
        - Save as: `11-wireshark-tcp-syn-scan-analysis.png`

**Security Detection**:

- Many SYN packets to different ports = port scan
- Many packets to same port = potential DoS
- Unusual protocols (ICMP tunneling) = C2 communication

---

### Phase 5: Wireless Security Configuration (60 minutes)

**Objective**: Compare WiFi encryption standards and configure secure SSID settings.

#### Task 5.1: Create Wireless Encryption Comparison

- [ ] **Step 1**: Document encryption standards
    
    ```markdown
    # Wireless Encryption Comparison
    
    | Standard | Year | Encryption | Key Length | Security Status |
    |----------|------|------------|------------|-----------------|
    | **WEP** | 1999 | RC4 | 40/104-bit | **DEPRECATED** - Crackable in <5 minutes |
    | **WPA** | 2003 | TKIP (RC4) | 128-bit | **DEPRECATED** - Vulnerable to attacks |
    | **WPA2** | 2004 | AES-CCMP | 128-bit | **Legacy** - Use for compatibility only |
    | **WPA3** | 2018 | AES-GCMP | 128/192-bit | **CURRENT** - Mandatory for new devices |
    
    ## Key Differences
    
    ### WEP (Wired Equivalent Privacy)
    - **Attack**: ARP replay injection → recover key in minutes
    - **Tools**: aircrack-ng with <100,000 packets
    - **Status**: Never use, even for guest networks
    
    ### WPA (WiFi Protected Access)
    - **Attack**: TKIP key collision attacks
    - **Improvement over WEP**: Per-packet key mixing
    - **Status**: Disabled by default in modern routers
    
    ### WPA2 (WiFi Protected Access 2)
    - **Attack**: KRACK (Key Reinstallation Attack) CVE-2017-13077
    - **Mitigation**: Patched in 2017, but WPA3 preferred
    - **Use Case**: Legacy device support (IoT cameras, printers)
    
    ### WPA3 (WiFi Protected Access 3)
    - **Key Feature**: SAE (Simultaneous Authentication of Equals)
      - Prevents offline dictionary attacks
      - Forward secrecy (past sessions stay encrypted even if password compromised)
    - **Modes**:
      - **WPA3-Personal**: For home/small business
      - **WPA3-Enterprise**: 192-bit encryption for government/military
    - **Status**: Mandatory for WiFi 6 certification
    
    ## Deployment Recommendations
    
    **Corporate Network**:
    - Primary SSID: WPA3-Enterprise (802.1X with RADIUS)
    - Guest SSID: WPA3-Personal (isolated VLAN)
    - Legacy SSID: WPA2-Personal (deprecated devices only, monitor for removal)
    
    **Healthcare (HIPAA)**:
    - Patient monitoring: WPA3-Enterprise + device certificates
    - Guest WiFi: Separate internet-only VLAN, WPA3-Personal
    - **Never** use WEP/WPA (HIPAA § 164.312 requires "industry standard" encryption)
    
    **Attack Mitigation**:
    - Disable WPS (PIN brute force vulnerability)
    - Hide SSID (security through obscurity, minimal benefit)
    - MAC filtering (easily spoofed, don't rely on this)
    - Use long passphrases (20+ characters)
    ```
    
- [ ] **Evidence**: Save as `12-wireless-encryption-comparison-table.md`
    

#### Task 5.2: Configure Secure WiFi Settings

- [ ] **Step 2**: Create recommended configuration
    
    ```markdown
    # Recommended WiFi Security Configuration
    
    ## Primary Corporate SSID
    - **SSID Name**: CorpWiFi (or company name)
    - **Broadcast SSID**: Enabled (hiding provides minimal security)
    - **Security Mode**: WPA3-Enterprise (802.1X)
    - **Authentication**: RADIUS server (certificates preferred over passwords)
    - **Encryption**: AES-GCMP-256
    - **Band**: 5 GHz (less interference, higher throughput)
    - **Channel Width**: 80 MHz (WiFi 6) or 40 MHz (WiFi 5)
    - **PMF (Protected Management Frames)**: Required (prevents deauth attacks)
    
    ## Guest WiFi SSID
    - **SSID Name**: Guest-WiFi
    - **Security Mode**: WPA3-Personal
    - **Passphrase**: 24-character random (rotate quarterly)
    - **Isolation**: Client isolation enabled (prevent guest-to-guest attacks)
    - **VLAN**: Separate VLAN, internet-only access
    - **Bandwidth Limit**: 10 Mbps per client
    - **Captive Portal**: Terms of service agreement
    
    ## IoT/Legacy Device SSID (Temporary)
    - **SSID Name**: IoT-Legacy
    - **Security Mode**: WPA2-Personal (AES only, disable TKIP)
    - **Passphrase**: Strong (20+ characters)
    - **VLAN**: Isolated from corporate network
    - **Firewall**: Block all except required cloud services
    - **Monitoring**: Alert on new devices joining
    - **Sunset Date**: Migrate to WPA3-capable devices by Q4 2024
    
    ## Additional Security Controls
    
    **Disable Legacy Features**:
    - [x] WEP
    - [x] WPA (TKIP)
    - [x] WPS (WiFi Protected Setup)
    - [x] UPnP (Universal Plug and Play)
    
    **Enable Security Features**:
    - [x] PMF (802.11w) - Protects management frames
    - [x] Fast BSS Transition (802.11r) - Secure roaming
    - [x] Opportunistic Wireless Encryption (OWE) - Open network encryption
    
    **Monitoring**:
    - Deploy wireless IDS (e.g., Kismet, Aircrack-ng in monitor mode)
    - Alert on:
      - Rogue access points (unauthorized APs with similar SSID)
      - Deauthentication floods (WiFi jamming attack)
      - Weak encryption (WEP/WPA detected)
    ```
    
- [ ] **Evidence**:
    
    - Create screenshot of example WiFi router config (or mock config)
    - Save as: `13-wifi-ssid-security-config.png`

---

## Observations & Failure Modes

### Common Issues

#### Issue 1: Ping Shows "Request Timed Out"

**Cause**: Firewall blocking ICMP, host offline, or wrong IP  
**Solution**:

```cmd
# Verify IP configuration
ipconfig /all

# Check firewall (Windows)
netsh advfirewall firewall show rule name="File and Printer Sharing (Echo Request - ICMPv4-In)"

# Enable ICMP if needed
netsh advfirewall firewall set rule name="File and Printer Sharing (Echo Request - ICMPv4-In)" new enable=yes
```

#### Issue 2: Traceroute Shows All Asterisks (*)

**Cause**: Routers configured to not respond to TTL-exceeded messages  
**Solution**:

- Asterisks are normal for some hops (security policy)
- Use `tracert -d` (don't resolve hostnames) to speed up
- Alternative: `pathping` on Windows (combines ping + tracert)

#### Issue 3: nmap Returns "Permission Denied"

**Cause**: Raw socket access requires root/admin  
**Solution**:

```bash
# Linux: Use sudo
sudo nmap -sS 192.168.0.0/24

# Windows: Run Command Prompt as Administrator
```

#### Issue 4: Wireshark Shows Encrypted WiFi Traffic

**Cause**: WPA2/WPA3 encryption enabled (expected)  
**Solution**:

```plaintext
# To decrypt WiFi in Wireshark:
Edit → Preferences → Protocols → IEEE 802.11
Enable decryption → Add Key:
  Key Type: wpa-pwd
  Key: password:SSID (e.g., SecurePass123:CorpWiFi)
  
# Note: Capture must include 4-way handshake
```

#### Issue 5: tcpdump Captures No Packets

**Cause**: Wrong interface selected  
**Solution**:

```bash
# List available interfaces
ip link show
# Or
tcpdump -D

# Capture on correct interface (e.g., eth0, wlan0)
sudo tcpdump -i eth0
```

---

## Analysis & Reflection

### Troubleshooting Methodology (CompTIA Approach)

**Step 1: Identify the Problem**

- Gather information (user complaint, error messages)
- Question the user (what changed recently?)
- Determine scope (single user vs. department-wide)

**Step 2: Establish Theory of Probable Cause**

- Use OSI model (bottom-up or top-down)
- List most likely causes first

**Step 3: Test Theory**

```
Layer 1 (Physical): Check cable, link lights
  └─> Tool: Visual inspection
Layer 2 (Data Link): Check MAC, ARP, switch port
  └─> Tool: arp -a, show mac address-table
Layer 3 (Network): Check IP, routing, ICMP
  └─> Tool: ping, tracert, route print
Layer 4 (Transport): Check TCP/UDP ports
  └─> Tool: telnet, nmap
Layer 5-7 (Application): Check DNS, HTTP, etc.
  └─> Tool: nslookup, curl, web browser
```

**Step 4: Establish Plan of Action**

- Document steps
- Consider impact (change window)
- Get approval if affects production

**Step 5: Implement Solution**

- Apply fix
- Monitor for side effects

**Step 6: Verify Functionality**

- Test with user
- Verify related systems still work

**Step 7: Document Findings**

- Root cause
- Solution applied
- Preventive measures

### Tool Selection Matrix

|Problem Type|Layer|Primary Tool|Secondary Tool|
|---|---|---|---|
|No network access|L1-L3|ping, ipconfig|tracert, route|
|Slow performance|L1-L7|Wireshark, pathping|nmap, iperf|
|DNS not resolving|L7|nslookup, dig|ipconfig /flushdns|
|Port closed|L4|telnet, nmap|netstat, ss|
|Rogue device|L2|nmap, arp|MAC address lookup|
|WiFi interference|L1|WiFi analyzer app|Channel change|
|Security breach|All|Wireshark, logs|IDS alerts, SIEM|

### Wireless Security Best Practices

**Frequency Selection**:

```
2.4 GHz:
  + Longer range (penetrates walls better)
  + Better legacy device support
  - Crowded (14 channels, only 3 non-overlapping: 1, 6, 11)
  - Interference (microwave ovens, Bluetooth, cordless phones)

5 GHz:
  + 24 non-overlapping channels (less interference)
  + Higher throughput (up to 1.3 Gbps on WiFi 5)
  - Shorter range (absorbed by walls)
  - Fewer legacy devices support

Recommendation: Dual-band (2.4 + 5 GHz), let clients choose
```

**Channel Planning**:

```
Office with 3 APs:
  AP1: Channel 1 (2.4 GHz) + Channel 36 (5 GHz)
  AP2: Channel 6 (2.4 GHz) + Channel 48 (5 GHz)
  AP3: Channel 11 (2.4 GHz) + Channel 149 (5 GHz)

Ensures no co-channel interference
```

**802.11 Standards Comparison**:

|Standard|Year|Max Speed|Frequency|Range|Use Case|
|---|---|---|---|---|---|
|802.11b|1999|11 Mbps|2.4 GHz|150 ft|Legacy only|
|802.11g|2003|54 Mbps|2.4 GHz|150 ft|Legacy only|
|802.11n (WiFi 4)|2009|600 Mbps|2.4/5 GHz|230 ft|Minimum today|
|802.11ac (WiFi 5)|2014|1.3 Gbps|5 GHz|230 ft|Current standard|
|802.11ax (WiFi 6)|2019|9.6 Gbps|2.4/5 GHz|230 ft|High-density|
|802.11ax (WiFi 6E)|2020|9.6 Gbps|6 GHz|200 ft|Future-proof|

**WiFi 6 (802.11ax) Improvements**:

- **OFDMA**: Multiple clients share channel simultaneously (vs. one at a time)
- **1024-QAM**: Higher data encoding (vs. 256-QAM in WiFi 5)
- **Target Wake Time**: IoT devices sleep longer (better battery)
- **BSS Coloring**: Reduces interference between overlapping networks

### Real-World Applications

**SOC Analyst Perspective**:

- **nmap scans** in logs = potential reconnaissance phase of attack
- **tcpdump** captures suspicious traffic for forensic analysis
- **Wireshark** decrypts protocols to identify data exfiltration

**Incident Response Workflow**:

1. Alert: Rogue AP detected on network
2. Use nmap to find rogue AP IP/MAC
3. Use Wireshark to capture rogue AP traffic
4. Analyze: Is it attacker (evil twin) or employee hotspot?
5. Locate physically using WiFi signal strength
6. Disconnect and investigate device

**Example: Evil Twin Attack Detection**:

```bash
# Scan for APs with same SSID
sudo airodump-ng wlan0

# Suspicious: Two "CorpWiFi" SSIDs with different MACs
# Legitimate: 00:11:22:33:44:55 (known AP)
# Rogue: AA:BB:CC:DD:EE:FF (unknown MAC)

# Capture handshake from rogue AP
sudo airodump-ng -c 6 --bssid AA:BB:CC:DD:EE:FF -w evil-twin wlan0

# Analyze in Wireshark for attacker patterns
```

### Time Tracking

|Phase|Estimated|Actual|Notes|
|---|---|---|---|
|Phase 1: Connectivity|60 min|___ min||
|Phase 2: DNS|30 min|___ min||
|Phase 3: Scanning|60 min|___ min||
|Phase 4: Packet Analysis|30 min|___ min||
|Phase 5: Wireless Security|60 min|___ min||
|**Total**|**240 min**|**___ min**||

---

## Exam Relevance Notes

### Exam Scenario Mapping

**N10-009 Performance-Based Questions (PBQs)**:

- **"User cannot access internet - troubleshoot"**
    - Steps: ping gateway → ping 8.8.8.8 → nslookup google.com
    - Identify layer where failure occurs
- **"Configure secure WiFi for office"**
    - Given: Router config screen
    - Action: Select WPA3, disable WPS, set strong passphrase

**Multiple Choice Question Patterns**:

- "Which tool traces the path packets take?" → traceroute/tracert
- "What is the most secure WiFi encryption?" → WPA3
- "Which nmap flag performs a SYN scan?" → `-sS`
- "What protocol does ping use?" → ICMP

### Key Exam Concepts Reinforced

**Troubleshooting Tools**:

- **ping**: ICMP echo request/reply (Layer 3 connectivity)
- **traceroute**: TTL-exceeded messages (path discovery)
- **nslookup/dig**: DNS queries (name resolution)
- **nmap**: Port scanning (host discovery, security auditing)
- **tcpdump**: CLI packet capture
- **Wireshark**: GUI packet analysis (protocol decode)

**WiFi Security**:

- **WEP**: Broken (RC4 key recovery)
- **WPA**: Deprecated (TKIP vulnerable)
- **WPA2**: Legacy (AES-CCMP, KRACK attack)
- **WPA3**: Current (SAE, forward secrecy, 192-bit enterprise)

**802.11 Standards**:

- **802.11n**: WiFi 4 (600 Mbps, 2.4/5 GHz)
- **802.11ac**: WiFi 5 (1.3 Gbps, 5 GHz only)
- **802.11ax**: WiFi 6 (9.6 Gbps, OFDMA, better for dense environments)

---

## Submission Checklist

Before marking this lab complete, verify:

- [ ] All 13 evidence files collected and named correctly
- [ ] Screenshots show Wireshark packet analysis
- [ ] Wireless encryption comparison table completed
- [ ] Troubleshooting decision tree flowchart created
- [ ] Tool selection matrix documented
- [ ] Time tracking completed
- [ ] Evidence files moved to `/mnt/user-data/outputs`

**Final Evidence Count**: 13 files  
**Documentation Pages**: 1 (this lab markdown)  
**Ready for GitHub**: Yes / No

---

## Next Steps

After completing this lab, you should:

1. **Review**: Study CompTIA Network+ troubleshooting methodology (7 steps)
2. **Practice**: Set up WiFi Pineapple (authorized penetration testing tool)
3. **Advance**: Complete Network+ certification exam
4. **Optional Challenge**: Deploy enterprise RADIUS server for WPA3-Enterprise

**Additional Resources**:

- [Wireshark User Guide](https://www.wireshark.org/docs/wsug_html_chunked/)
- [nmap Network Scanning Book](https://nmap.org/book/)
- [WiFi Alliance: WPA3 Specification](https://www.wi-fi.org/discover-wi-fi/security)
- CompTIA Network+ N10-009 Official Study Guide, Chapter 5 (Troubleshooting)

---

## Appendix: Troubleshooting Decision Tree

```
User Reports Network Issue
        |
        v
Can user ping 127.0.0.1? (localhost)
    |
    +-- NO --> NIC/driver issue → Check Device Manager
    |
    +-- YES --> Can ping default gateway?
                    |
                    +-- NO --> Layer 1/2 issue
                    |          └─> Check cable, switch port, IP config
                    |
                    +-- YES --> Can ping 8.8.8.8?
                                    |
                                    +-- NO --> Routing/firewall issue
                                    |          └─> Check route table, firewall rules
                                    |
                                    +-- YES --> Can resolve google.com?
                                                    |
                                                    +-- NO --> DNS issue
                                                    |          └─> Check DNS servers, hosts file
                                                    |
                                                    +-- YES --> Application-layer issue
                                                               └─> Check app config, proxy, authentication
```