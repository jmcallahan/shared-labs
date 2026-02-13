---
title: "IPv4 Network Services: DHCP, DNS, and IPv6 Configuration"
difficulty: Intermediate
Cert:
  - Network+
  - NT10-009
Exam-Objectives:
  - Net+ 1.4
  - Net+ 3.4
created: 2026-02-13
completed:
time-hours: 3
complete: false
GitHub-ready: false
evidence-files: 10
notes:
tags:
  - dhcp
  - dns
  - ipv6
  - ipv4
  - slaac
  - cname-records
  - network-services
---
---
## Objective Mapping

**3.4 Given a scenario, implement IPv4 and IPv6 network services**

- Dynamic addressing (DHCP)
- Name resolution (DNS)
- IPv6 addressing methods (SLAAC)

**1.4 Given a scenario, configure a subnet and use appropriate IP addressing schemes**

- IPv6 concepts

## Scenario

You're a junior network administrator at a legal firm expanding from 30 to 100 employees. The current static IP addressing scheme is unmanageable, and DNS queries are handled by external resolvers (creating privacy concerns for client names). Your tasks:

1. **Deploy a DHCP server** to automate IP address assignment for new workstations
2. **Configure internal DNS** with CNAME records for service aliases (intranet.firm.local → dc01.firm.local)
3. **Enable IPv6** using SLAAC (Stateless Address Autoconfiguration) to future-proof the network

This lab simulates **essential enterprise network services** that SOC analysts must understand to troubleshoot connectivity issues and identify anomalies (rogue DHCP servers, DNS poisoning).

## Learning Objectives

- Install and configure Windows Server DHCP role
- Create DHCP scopes with IP ranges and options
- Verify DHCP leases on client machines
- Add DNS CNAME records for service aliases
- Enable IPv6 SLAAC on network interfaces
- Troubleshoot DHCP/DNS issues using `ipconfig` and `nslookup`

## Required Environment

### Hardware/Software

- **VMs**: ACIDC01 (Windows Server 2022 DC), ACIDM01 (Windows Server 2022 Member), ACIWIN11 (Windows 11 Pro)
- **Tools**: Server Manager, DHCP MMC, DNS Manager, Command Prompt

### Host System

- **OS**: Windows 11 Pro or Windows Server 2022
- **RAM**: 12GB minimum
- **Network**: Internal virtual network

## Deliverables

### Evidence Files (Required)

1. `01-aciwin11-ipconfig-before-dhcp.txt` – No DHCP lease initially
2. `02-acidm01-dhcp-role-installation.png` – Add Roles wizard installing DHCP
3. `03-dhcp-scope-192-168-0-100-150.png` – Scope configuration (100-150 range)
4. `04-dhcp-scope-options-gateway-dns.png` – Default gateway and DNS server options
5. `05-acidm01-dhcp-authorization.png` – DHCP server authorized in AD
6. `06-aciwin11-ipconfig-after-dhcp.txt` – Lease obtained from DHCP server
7. `07-dns-cname-record-intranet.png` – CNAME: intranet → acidc01.aciplab.com
8. `08-nslookup-intranet-alias-test.txt` – Resolves to DC IP address
9. `09-aciwin11-ipv6-slaac-enabled.png` – Link-local and global IPv6 addresses
10. `10-ipv6-ping-test-to-dc.txt` – Successful IPv6 ping to domain controller

### Documentation (Required)

- **DHCP scope planning document**: Explain IP range selection and reservations
- **DNS alias strategy**: List all CNAME records and purposes
- **IPv6 transition plan**: SLAAC vs. DHCPv6 comparison

## Lab Tasks

### Phase 1: DHCP Server Deployment (60 min)

#### Task 1.1: Verify Pre-DHCP Configuration

- [ ] On ACIWIN11:
    
    ```cmd
    ipconfig /all > C:\Evidence\01-aciwin11-ipconfig-before-dhcp.txttype C:\Evidence\01-aciwin11-ipconfig-before-dhcp.txt
    ```
    
- [ ] Verify `DHCP Enabled: No` for Ethernet 2 adapter
- [ ] **Evidence**: `01-aciwin11-ipconfig-before-dhcp.txt`

#### Task 1.2: Install DHCP Server Role

- [ ] On ACIDM01 → **Server Manager → Add Roles and Features**
    
    ```
    Installation Type: Role-basedServer: ACIDM01Server Roles: [x] DHCP ServerAdd Features → Next → Install
    ```
    
- [ ] **Screenshot**: `02-acidm01-dhcp-role-installation.png`

#### Task 1.3: Create DHCP Scope

- [ ] Open **DHCP MMC** (Tools → DHCP)
- [ ] Expand `acidm01.aciplab.com` → IPv4 → Right-click → **New Scope**
    
    ```
    Scope Name: Corporate_LANIP Address Range: 192.168.0.100 - 192.168.0.150Subnet Mask: 255.255.255.0 (/24)Exclusions: 192.168.0.100-192.168.0.110 (reserved for servers)Lease Duration: 8 days (default)
    ```
    
- [ ] **Screenshot**: `03-dhcp-scope-192-168-0-100-150.png`

#### Task 1.4: Configure Scope Options

- [ ] In New Scope Wizard → Configure DHCP Options
    
    ```
    Default Gateway (Router): 192.168.0.1DNS Servers: 192.168.0.10 (ACIDC01)DNS Domain Name: aciplab.com
    ```
    
- [ ] **Screenshot**: `04-dhcp-scope-options-gateway-dns.png`
- [ ] **Activate scope**

#### Task 1.5: Authorize DHCP Server in Active Directory

- [ ] In DHCP MMC → Right-click `acidm01.aciplab.com` → **Authorize**
- [ ] Verify green checkmark appears (server authorized)
- [ ] **Screenshot**: `05-acidm01-dhcp-authorization.png`

#### Task 1.6: Test DHCP Lease on Client

- [ ] On ACIWIN11:
    
    ```cmd
    ipconfig /releaseipconfig /renewipconfig /all > C:\Evidence\06-aciwin11-ipconfig-after-dhcp.txt
    ```
    
- [ ] Verify:
    - DHCP Enabled: Yes
    - IP Address: 192.168.0.111-150 range
    - Default Gateway: 192.168.0.1
    - DNS Servers: 192.168.0.10
- [ ] **Evidence**: `06-aciwin11-ipconfig-after-dhcp.txt`

---

### Phase 2: DNS CNAME Records (30 min)

#### Task 2.1: Create CNAME Record

- [ ] On ACIDC01 → **Server Manager → Tools → DNS**
- [ ] Expand `ACIDC01 → Forward Lookup Zones → aciplab.com`
- [ ] Right-click → **New Alias (CNAME)**
    
    ```
    Alias Name: intranetFQDN for Target Host: acidc01.aciplab.com
    ```
    
- [ ] Click OK
- [ ] **Screenshot**: `07-dns-cname-record-intranet.png`

#### Task 2.2: Test DNS Resolution

- [ ] On ACIWIN11:
    
    ```cmd
    nslookup intranet.aciplab.com > C:\Evidence\08-nslookup-intranet-alias-test.txttype C:\Evidence\08-nslookup-intranet-alias-test.txt
    ```
    
- [ ] Verify output shows:
    
    ```
    Server: acidc01.aciplab.comAddress: 192.168.0.10intranet.aciplab.com    canonical name = acidc01.aciplab.comName:    acidc01.aciplab.comAddress: 192.168.0.10
    ```
    
- [ ] **Evidence**: `08-nslookup-intranet-alias-test.txt`

---

### Phase 3: IPv6 SLAAC Configuration (60 min)

#### Task 3.1: Enable IPv6 on Windows 11 Client

- [ ] On ACIWIN11 → **Settings → Network & Internet → Ethernet → Properties**
    
    ```
    IPv6: Enabled (if disabled)
    ```
    
- [ ] Verify auto-configuration:
    
    ```cmd
    ipconfig /all | findstr "IPv6"
    ```
    
- [ ] **Screenshot**: `09-aciwin11-ipv6-slaac-enabled.png`
- [ ] Expected output:
    
    ```
    Link-local IPv6 Address: fe80::xxxx:xxxx:xxxx:xxxx%12IPv6 Address (if router advertising): 2001:db8::xxxx (example)
    ```
    

#### Task 3.2: Test IPv6 Connectivity

- [ ] On ACIWIN11:
    
    ```cmd
    # Ping domain controller using IPv6 link-local addressping -6 acidc01.aciplab.com > C:\Evidence\10-ipv6-ping-test-to-dc.txttype C:\Evidence\10-ipv6-ping-test-to-dc.txt
    ```
    
- [ ] **Evidence**: `10-ipv6-ping-test-to-dc.txt`

**Note**: If no IPv6 router advertisements present, only link-local addresses (fe80::) will be assigned. This is expected in isolated lab environments.

---

## Observations & Failure Modes

### Issue 1: DHCP Server Not Assigning Leases

**Cause**: Server not authorized in Active Directory  
**Solution**:

```cmd
# On ACIDM01 (as admin)
netsh dhcp server \\acidm01 add server acidm01.aciplab.com 192.168.0.11
```

Verify in DHCP MMC (green checkmark on server)

### Issue 2: Client Gets APIPA Address (169.254.x.x)

**Cause**: Cannot reach DHCP server  
**Solution**:

```cmd
# Check firewall
netsh advfirewall firewall show rule name="DHCP Server"

# Verify scope is activated
# DHCP MMC → Scope → Status: Active (green arrow)
```

### Issue 3: nslookup Returns NXDOMAIN for CNAME

**Cause**: DNS zone not updated or typo in alias name  
**Solution**:

- Clear DNS cache: `ipconfig /flushdns`
- Verify CNAME in DNS Manager
- Check DNS server in `ipconfig /all` matches ACIDC01 IP

### Issue 4: No IPv6 Address Assigned (Only Link-Local)

**Cause**: No router advertisements (normal in isolated lab)  
**Solution**:

- This is expected if no IPv6 router configured
- Link-local (fe80::) addresses still allow local IPv6 communication
- For testing, manually assign: `netsh interface ipv6 add address "Ethernet" 2001:db8::10/64`

---

## Analysis & Reflection

### DHCP Scope Planning

**Best Practices**:

```markdown
| Subnet | Range | Purpose | Exclusions |
|--------|-------|---------|------------|
| 192.168.0.0/24 | .100-.150 | DHCP pool | .1-.99 (static servers) |
| 192.168.0.0/24 | .200-.250 | VoIP phones | .151-.199 (printers) |

**Reservations**: MAC-based assignments for:
- Printers
- Network cameras
- APs
```

### DNS CNAME Strategy

**Common Aliases**:

```
intranet.firm.local → dc01.firm.local (SharePoint)
mail.firm.local → exchange01.firm.local
vpn.firm.local → vpn-gateway.firm.local
```

**Security Note**: CNAMEs can be used in phishing (www.bankofamerica.com → attacker.com). Monitor DNS logs.

### IPv6 Transition Considerations

|Method|Description|Use Case|
|---|---|---|
|**SLAAC**|Auto-config via router advertisements|Simple networks, IoT|
|**DHCPv6**|Centralized management like DHCPv4|Enterprise, need logging|
|**Dual Stack**|Run IPv4 + IPv6 simultaneously|Transition period|

### Real-World Applications

**SOC Analyst Perspective**:

- **Rogue DHCP**: Attacker adds unauthorized DHCP server → clients get wrong gateway → MitM attack
- **DNS Poisoning**: Altered CNAME records redirect users to phishing sites
- **DHCP Starvation**: DoS attack exhausts DHCP pool

**Detection**:

```cmd
# Show DHCP leases
Get-DhcpServerv4Lease -ComputerName acidm01 -ScopeId 192.168.0.0

# Monitor DNS queries (SIEM log source)
Get-DnsServerStatistics
```

### Time Tracking

|Phase|Estimated|Actual|
|---|---|---|
|Phase 1: DHCP|60 min|___ min|
|Phase 2: DNS|30 min|___ min|
|Phase 3: IPv6|60 min|___ min|
|**Total**|**150 min**|**___ min**|

---

## Exam Relevance Notes

**N10-009 Key Concepts**:

- **DHCP DORA Process**: Discover → Offer → Request → Acknowledge
- **DNS Record Types**:
    - A: Name → IPv4
    - AAAA: Name → IPv6
    - CNAME: Alias → Canonical name
    - MX: Mail exchange
    - PTR: Reverse lookup
- **IPv6 Addressing**:
    - Link-local: fe80::/10
    - Global unicast: 2000::/3
    - Multicast: ff00::/8

**PBQ Scenarios**:

- "Configure DHCP scope for 192.168.1.0/24, exclude .1-.10"
- "Add CNAME record pointing www to web01.company.com"

---

## Submission Checklist

- [ ] All 10 evidence files collected
- [ ] DHCP scope planning document completed
- [ ] DNS alias strategy documented
- [ ] IPv6 transition plan analysis included
- [ ] Time tracking updated

**Final Evidence Count**: 10 files  
**Ready for GitHub**: Yes / No

---

## Next Steps

1. **Review**: DHCPv6 vs. SLAAC
2. **Practice**: Configure DHCP relay for multi-subnet environment
3. **Advance**: Proceed to Lab 06 (Software-Defined Networking)
4. **Challenge**: Set up DNSSEC for secure DNS resolution