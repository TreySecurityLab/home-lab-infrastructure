# Hardware Inventory

## Documentation Status

**Authoritative inventory revision:** August 20, 2026  
**Purpose:** Record the physical systems, virtualization roles, baseline workload memory allocation, and authoritative management/workload addressing used by the home lab.

This page reflects the updated hardware inventory plus the authoritative VLAN/host designation source and verified management-network evidence. Specifications not present in those sources—such as CPU model, storage capacity, NIC model, exact Proxmox bridge names, and VM vCPU allocations—are intentionally not invented.

## Physical Systems

| Hardware / Hostname | Platform / RAM | Purpose |
|---|---|---|
| `opnsense-fw` | Bare-metal OPNsense / 8 GB RAM | ISP-facing edge firewall/router providing NAT, lab VLAN routing, policy enforcement, VPN capability, and controlled Internet connectivity |
| `SW-Lab-01` | Aruba J9774A managed switch | Wired connectivity, VLAN segmentation, 802.1Q tagging/trunking, access-port assignment, and traffic mirroring |
| `ENTHOST-01` | 32 GB desktop / Proxmox VE | Hosts enterprise server and workstation VMs |
| `SECHOST-01` | 32 GB Ryzen laptop / Proxmox VE | Hosts SIEM/XDR, network detection, DFIR, and threat-hunting workloads |
| `MGMT-01` | 16 GB laptop | Dedicated administration system for OPNsense, SW-Lab-01, Proxmox, servers, and other infrastructure |
| `MGMT-BACKUP` | Laptop | Backup/remote administration system; local VLAN 10 address 10.10.10.10/24 when physically connected |
| `READTEAM-01` | 16 GB laptop / Kali Linux bare metal | Dedicated offensive-security system for scanning, enumeration, attack simulation, exploitation exercises, and defensive-control validation |

**WAN placement:** OPNsense is the first lab routing/security device on the ISP-facing connection. No separate consumer router is part of the home-lab infrastructure.

## Authoritative Host / IP Assignments

| VLAN | Host / inventory label | Address | Purpose / status |
|---:|---|---|---|
| 10 MANAGEMENT | `opnsense-fw` / `OPNsense-FW` | 10.10.10.1 | OPNsense management gateway |
| 10 MANAGEMENT | `SW-Lab-01` / `SW-LAB-01` | 10.10.10.2 | Switch management |
| 10 MANAGEMENT | `ENTHOST-01` | 10.10.10.3 | Enterprise Proxmox management |
| 10 MANAGEMENT | `SECHOST-01` | 10.10.10.4 | Security Proxmox management |
| 10 MANAGEMENT | `MGMT-01` | 10.10.10.6 | Management Host |
| 10 MANAGEMENT | `MGMT-BACKUP` | 10.10.10.10 | Backup management laptop when locally connected |
| 20 USERS | `WIN11-01` | 10.10.20.10 | Windows 11 workstation |
| 30 DMZ | `WEB-01` | 10.10.30.10 | DMZ web server |
| 50 SERVERS | `DC-01` | 10.10.50.10 | Primary domain-controller workload |
| 50 SERVERS | `DC-02` | 10.10.50.20 | Planned/rotational second domain controller; approximately 3–4 GB RAM when active |
| 50 SERVERS | `FILE-01` | 10.10.50.30 | File server |
| 50 SERVERS | `LINUX-01` | 10.10.50.40 | Ubuntu/Linux server |
| 60 REDTEAM | `READTEAM-01` | 10.10.60.10 | Bare-metal red-team host |

`DC-02` is intentionally part of the planned home lab as a rotational second domain controller on VLAN 50. Its address is authoritative and its recommended RAM is approximately 3–4 GB. It is not part of the normal always-on 22 GB enterprise guest allocation unless the architecture is later revised and re-verified.

## Network Placement

| System | Network placement |
|---|---|
| `MGMT-01` | VLAN 10 — MANAGEMENT; physically attached to SW-Lab-01 port 8 |
| `MGMT-BACKUP` | VLAN 10 — MANAGEMENT when locally connected; physically attached to SW-Lab-01 port 6 |
| `opnsense-fw` | Direct ISP-facing WAN; Layer-3 gateways for VLANs 10/20/30/40/50/60; SW-Lab-01 port 1 trunk |
| `SW-Lab-01` | Management on VLAN 10; Layer-2 transport for VLANs 10/20/30/40/50/60 |
| `ENTHOST-01` | Physically attached to SW-Lab-01 port 7; Proxmox management on VLAN 10; guests planned on VLANs 20, 30, and 50 |
| `SECHOST-01` | Proxmox management on VLAN 10; security workloads on VLAN 40; passive sensor path planned separately |
| `READTEAM-01` | VLAN 60 — REDTEAM |

## Enterprise Virtualization Host

**Platform:** `ENTHOST-01`, 32 GB desktop running Proxmox VE.  
**Role:** Provide the virtualized enterprise systems required for Active Directory, Windows endpoint management, Linux administration, web services, file services, and security testing.

| VM | Recommended RAM | Primary function | VLAN | Address | Purpose in the home lab |
|---|---:|---|---|---|---|
| `DC-01` — Windows Server | 4 GB | Active Directory Domain Services + DNS | VLAN 50 | 10.10.50.10 | Lab domain, centralized authentication, directory services, Group Policy testing, and AD-integrated DNS |
| `LINUX-01` — Ubuntu Server | 4 GB | Linux server | VLAN 50 | 10.10.50.40 | Linux administration, SSH, services, hardening, logging, automation, and security exercises |
| `WEB-01` — Web Server | 2 GB | DMZ web application/server | VLAN 30 | 10.10.30.10 | Isolated web service for hardening, monitoring, and controlled attack/defense exercises |
| `WIN11-01` — Windows 11 Workstation | 8 GB | Domain-joined user endpoint | VLAN 20 | 10.10.20.10 | Active Directory, Group Policy, endpoint logging, Sysmon, Wazuh, and red/blue-team exercises |
| `FILE-01` — File Server | 4 GB | Centralized file services | VLAN 50 | 10.10.50.30 | SMB/file shares, NTFS permissions, access testing, audit logging, and centralized storage exercises |
| `DC-02` — Windows Server | 3–4 GB | Second domain controller + DNS | VLAN 50 | 10.10.50.20 | Planned/rotational second DC to be deployed after `DC-01` is stable |

### Enterprise Host Memory Plan

The authoritative baseline remains the five normal enterprise VMs:

| Allocation | RAM |
|---|---:|
| Always-on baseline guest VM allocation | 22 GB |
| Reserved for Proxmox host | 4 GB |
| Unallocated baseline headroom | 6 GB |
| Physical RAM | 32 GB |
| `DC-02` rotational allocation | Approximately 3–4 GB additional only while active |

The normal 22 GB guest allocation does **not** include `DC-02`. When `DC-02` is powered on for second-domain-controller exercises, its approximately 3–4 GB is drawn from available headroom or by rotating another workload off. Do not treat `DC-02` as continuously running unless the architecture is later revised and re-verified.

## Security Virtualization Host

**Platform:** `SECHOST-01`, 32 GB Ryzen laptop running Proxmox VE.  
**Role:** Dedicated blue-team platform for centralized telemetry, SIEM/XDR, network intrusion detection, protocol analysis, threat hunting, and digital forensics.

| Security VM | Recommended RAM | Core tools / role | VLAN | Purpose |
|---|---:|---|---|---|
| Wazuh SIEM/XDR VM | 8 GB | Wazuh manager, indexer, dashboard | VLAN 40 — SECOPS | Centralized telemetry, alert correlation, FIM, security configuration assessment, vulnerability visibility, and investigation |
| Network Security Monitoring VM | 8 GB | Suricata + Zeek | VLAN 40 — SECOPS + passive sensor path | IDS inspection and network/protocol metadata from mirrored traffic |
| DFIR / Threat Hunting VM | 4 GB | Velociraptor / investigation tools | VLAN 40 — SECOPS | Endpoint visibility, artifact collection, live response, triage, threat hunting, and DFIR |

### Security Host Memory Plan

| Allocation | RAM |
|---|---:|
| Guest VM allocation | 20 GB |
| Reserved for Proxmox host | 4 GB |
| Unallocated headroom | 8 GB |
| Physical RAM | 32 GB |

## Security Tooling Notes

Wazuh is the baseline central SIEM/XDR platform. Suricata and Zeek share the Network Security Monitoring VM, while Velociraptor remains separate for DFIR and threat hunting. Splunk or Security Onion may be introduced later as rotational training workloads rather than always-on baseline VMs.

## Source Basis

- *Home Lab Hardware Inventory — UPDATED* — physical systems, VM roles, and baseline memory plans.
- *Authoritative VLAN Designations — final* — authoritative hostnames/inventory labels and IP assignments.
- *Home Lab Network Topology Overview — UPDATED* — direct ISP placement, VLAN placement, virtualization roles, and telemetry flow.
- *OPNsense Home Lab Configuration — UPDATED* — direct ISP WAN and current firewall/switch interface baseline.
- *MGMT-01 Remote RDP Resolution Summary* — verified port 6 VLAN 10 assignment and MGMT-BACKUP local management address.
