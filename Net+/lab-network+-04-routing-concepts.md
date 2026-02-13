---
title: "Routing Concepts: Static Routes, NAT, and pfSense Subinterfaces"
difficulty: Intermediate
Cert:
  - Network+
Exam-Objectives:
  - Net+ 2.1
  - Net+ 3.1
  - Net+ 3.2
  - Net+ 2.3
created: 2026-02-13
completed:
time-hours: 3
complete: false
GitHub-ready: false
evidence-files: 10
notes:
tags:
  - network-routing
  - static-routes
  - nat
  - port-forwarding
  - pfsense
  - subinterfaces
  - vlan-routing
---
---
## Objective Mapping

**2.1 Compare and contrast various devices, their features, and their appropriate placement on the network**

- Router
- Firewall
- Network function virtualization (NFV)

**2.3 Given a scenario, configure and deploy common Ethernet switching features**

- Interface configurations (subinterfaces)

**3.1 Given a scenario, use the appropriate IPv4 network addressing**

- Static routing

**3.2 Given a scenario, configure services relating to routing**

- NAT/PAT

## Scenario

You're a network security analyst supporting a growing e-commerce company. The infrastructure team is migrating from a flat network to a **segmented multi-VLAN environment** with pfSense as the central router/firewall. Your responsibilities include:

1. **Configuring static routes** to enable communication between isolated network segments
2. **Implementing NAT/port forwarding** to expose an internal FTP server to external vendors
3. **Setting up VLAN subinterfaces** on pfSense to route between VLANs (marketing and engineering departments)

This lab mirrors real-world scenarios where **Layer 3 routing and NAT are critical for both security and connectivity**. It prepares you for SOC analyst roles where you must troubleshoot routing issues and secure network perimeters.

## Learning Objectives

- Configure pfSense WAN firewall rules for outbound traffic
- Create LAN interfaces with static IP assignments
- Add routing gateways and static routes in pfSense
- Implement NAT port forwarding for FTP services
- Configure VLAN subinterfaces for inter-VLAN routing
- Apply explicit deny firewall rules for security hardening
- Troubleshoot routing table issues using `route print`

## Required Environment

### Hardware/Software

- **VMs**: ACIDC01 (Windows Server 2022), ACIWIN11 (Windows 11), ACIPFSENSE (pfSense 2.7.2)
- **Tools**: pfSense WebGUI, Windows Command Prompt, Microsoft Edge
- **Network**: Virtualized environment with multiple interfaces

### Host System

- **OS**: Windows 11 Pro or Windows Server 2022
- **RAM**: 8GB minimum for all VMs
- **Hypervisor**: VMware Workstation Pro or Hyper-V

## Deliverables

### Evidence Files (Required)

1. `01-pfsense-firewall-wan-rule-creation.png` – WAN firewall rule allowing outbound
2. `02-pfsense-lan-interface-marketing-created.png` – Marketing LAN (172.16.0.1/24)
3. `03-pfsense-routing-gateway-marketing.png` – Gateway configuration
4. `04-pfsense-static-route-to-10-network.png` – Static route via Marketing gateway
5. `05-acidc01-route-print-updated.txt` – Routing table showing new route
6. `06-pfsense-nat-port-forward-ftp-rule.png` – FTP port 21 forwarding to 192.168.0.22
7. `07-pfsense-vlan-10-subinterface.png` – VLAN 10 subinterface configuration
8. `08-pfsense-vlan-20-subinterface.png` – VLAN 20 subinterface configuration
9. `09-pfsense-firewall-vlan-10-explicit-deny.png` – Deny rule blocking VLAN 10 → VLAN 20
10. `10-vlan-isolation-test-output.txt` – Ping test showing blocked traffic

### Documentation (Required)

- **Network diagram**: Show all subnets, interfaces, and routing paths
- **Routing table analysis**: Explain how packets are forwarded
- **NAT configuration notes**: Port forwarding security implications

## Lab Tasks

### Phase 1: Static Routing Configuration (60 min)

#### Task 1.1: Configure WAN Firewall Rule

- [ ] Connect to ACIWIN11 → Open Microsoft Edge → Navigate to `http://192.168.0.5`
- [ ] Login to pfSense: `admin / Passw0rd`
- [ ] **Firewall → Rules → WAN** → Add rule (top of list)
    
    ```
    Action: PassProtocol: AnySource: WAN netDestination: AnyDescription: Allow outbound WAN traffic
    ```
    
- [ ] **Screenshot**: `01-pfsense-firewall-wan-rule-creation.png`

#### Task 1.2: Create Marketing LAN Interface

- [ ] **Interfaces → Assignments** → Add `em1` as `OPT1`
- [ ] Rename OPT1 to **MARKETING**
- [ ] Configure:
    
    ```
    IPv4 Configuration: StaticIPv4 Address: 172.16.0.1/24Description: Marketing Department LAN
    ```
    
- [ ] **Screenshot**: `02-pfsense-lan-interface-marketing-created.png`

#### Task 1.3: Add Routing Gateway

- [ ] **System → Routing → Gateways** → Add
    
    ```
    Name: Marketing_GWInterface: MARKETINGGateway: 172.16.0.2Description: Gateway to 10.0.0.0/24 network
    ```
    
- [ ] **Screenshot**: `03-pfsense-routing-gateway-marketing.png`

#### Task 1.4: Create Static Route

- [ ] **System → Routing → Static Routes** → Add
    
    ```
    Destination Network: 10.0.0.0/24Gateway: Marketing_GW (172.16.0.2)Description: Route to internal 10.x network
    ```
    
- [ ] **Screenshot**: `04-pfsense-static-route-to-10-network.png`

#### Task 1.5: Verify Routing Table on Windows Client

- [ ] On ACIDC01, open Command Prompt:
    
    ```cmd
    route print > C:\Evidence\05-acidc01-route-print-updated.txttype C:\Evidence\05-acidc01-route-print-updated.txt
    ```
    
- [ ] Verify route to `10.0.0.0/24` exists via `172.16.0.2`
- [ ] **Evidence**: `05-acidc01-route-print-updated.txt`

---

### Phase 2: NAT/Port Forwarding (40 min)

#### Task 2.1: Configure FTP Port Forwarding

- [ ] **Firewall → NAT → Port Forward** → Add rule
    
    ```
    Protocol: TCPDestination Port: FTP (21)Redirect Target IP: 192.168.0.22Redirect Target Port: FTP (21)Description: FTP Port FWD to internal serverFilter Rule Association: Pass
    ```
    
- [ ] Apply Changes
- [ ] **Screenshot**: `06-pfsense-nat-port-forward-ftp-rule.png`

#### Task 2.2: Test Port Forwarding

- [ ] On ACIWIN11:
    
    ```cmd
    # Test FTP connection (will fail if 192.168.0.22 doesn't exist)telnet [WAN-IP] 21# Expected: Connection attempt redirected to 192.168.0.22
    ```
    
- [ ] Document behavior in lab notes

---

### Phase 3: VLAN Subinterfaces and Inter-VLAN Routing (80 min)

#### Task 3.1: Create VLAN 10 Subinterface

- [ ] **Interfaces → Assignments → VLANs** → Add
    
    ```
    Parent Interface: em2VLAN Tag: 10Description: Engineering VLAN
    ```
    
- [ ] Assign VLAN 10 to new interface (OPT2)
- [ ] Configure interface:
    
    ```
    IPv4 Address: 192.168.10.1/24Description: Engineering Department
    ```
    
- [ ] **Screenshot**: `07-pfsense-vlan-10-subinterface.png`

#### Task 3.2: Create VLAN 20 Subinterface

- [ ] **Interfaces → Assignments → VLANs** → Add
    
    ```
    Parent Interface: em2VLAN Tag: 20Description: Marketing VLAN
    ```
    
- [ ] Assign and configure:
    
    ```
    IPv4 Address: 192.168.20.1/24Description: Marketing Department
    ```
    
- [ ] **Screenshot**: `08-pfsense-vlan-20-subinterface.png`

#### Task 3.3: Create Explicit Deny Rule

- [ ] **Firewall → Rules → VLAN 10** → Add rule (top)
    
    ```
    Action: BlockProtocol: AnySource: VLAN 10 netDestination: VLAN 20 netDescription: Explicit deny VLAN 10 to VLAN 20
    ```
    
- [ ] **Screenshot**: `09-pfsense-firewall-vlan-10-explicit-deny.png`

#### Task 3.4: Test VLAN Isolation

- [ ] From a client in VLAN 10:
    
    ```cmd
    ping 192.168.20.10# Should FAIL due to firewall rule
    ```
    
- [ ] Export results: `10-vlan-isolation-test-output.txt`

---

## Observations & Failure Modes

### Issue 1: Static Route Not Appearing in Windows Routing Table

**Cause**: pfSense static route exists but Windows doesn't know about it  
**Solution**: Add route manually on Windows client:

```cmd
route add 10.0.0.0 mask 255.255.255.0 172.16.0.1
```

### Issue 2: Port Forwarding Not Working

**Cause**: Firewall rule auto-creation disabled  
**Solution**: Ensure "Filter Rule Association" is set to "Pass" when creating NAT rule

### Issue 3: VLANs Can Still Communicate Despite Deny Rule

**Cause**: Rule order - permit rules above deny rules  
**Solution**: Move deny rule to **top** of firewall rules list

---

## Analysis & Reflection

### Security Considerations

**Static Routes**:

- **Risk**: Misconfigured routes can bypass firewalls
- **Mitigation**: Validate all static routes with `tracert`

**NAT/Port Forwarding**:

- **Risk**: Exposes internal services to internet
- **Mitigation**: Use strong authentication, log all access, limit source IPs

**VLAN Isolation**:

- **Pro**: Prevents lateral movement in breaches
- **Con**: Requires Layer 3 device to route between VLANs

### Real-World Applications

**SOC Analyst Use Case**:

- Monitor NAT logs for port scanning attempts
- Verify VLAN ACLs block unauthorized inter-VLAN traffic
- Use routing tables to identify suspicious routes added by malware

### Time Tracking

|Phase|Estimated|Actual|
|---|---|---|
|Phase 1: Static Routing|60 min|___ min|
|Phase 2: NAT|40 min|___ min|
|Phase 3: VLAN Subinterfaces|80 min|___ min|
|**Total**|**180 min**|**___ min**|

---

## Exam Relevance Notes

**N10-009 Key Concepts**:

- **Static vs. Dynamic Routing**: Static requires manual config, dynamic uses protocols (OSPF, EIGRP)
- **NAT Types**: SNAT (source NAT), DNAT (destination NAT), PAT (port address translation)
- **Subinterfaces**: Allow multiple VLANs on single physical interface (router-on-a-stick)

**PBQ Scenarios**:

- "Configure port forwarding for web server on port 80"
- "Create static route to 10.1.1.0/24 via 192.168.1.1"

---

## Submission Checklist

- [ ] All 10 evidence files collected
- [ ] Network diagram shows subnets and routing paths
- [ ] NAT configuration documented with security notes
- [ ] Routing table analysis completed
- [ ] Time tracking updated

**Final Evidence Count**: 10 files  
**Ready for GitHub**: Yes / No

---

## Next Steps

1. **Review**: Dynamic routing protocols (OSPF, BGP)
2. **Practice**: Configure DHCP relay between VLANs
3. **Advance**: Proceed to Lab 05 (IPv4 Network Services)
4. **Challenge**: Add VPN subinterface with NAT traversal