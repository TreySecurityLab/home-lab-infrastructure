# Hardware Inventory

## Documentation Status

**Authoritative inventory date:** August 16, 2026  
**Purpose:** Record the physical systems, virtualization roles, and baseline workload memory allocation used by the home lab.

This page reflects the current hardware inventory source. Specifications that are not present in the source—such as CPU model, storage capacity, NIC model, and exact Proxmox node hostname—are intentionally not invented.

## Physical Systems

| Hardware / Hostname | Platform / RAM | Purpose |
|---|---|---|
| Archer AXE75 | Wi-Fi router | Provides the home network Wi-Fi service and upstream Internet connection used by the lab. OPNsense connects downstream from this router. |
| `opnsense-fw` | Bare-metal OPNsense / 8 GB RAM | Provides perimeter firewalling, NAT, lab VLAN routing, policy enforcement, and controlled connectivity between the lab and the upstream home network. |
| `SW-Lab-01` | Aruba J9774A managed switch | Provides wired connectivity, VLAN segmentation, 802.1Q tagging/trunking, access-port assignment, and traffic mirroring for security monitoring. |
| Enterprise Virtualization Host | 32 GB desktop / Proxmox VE | Hosts the primary server and workstation VMs used to simulate a small enterprise environment. |
| Security Virtualization Host | 32 GB Ryzen laptop / Proxmox VE | Hosts centralized security monitoring, SIEM/XDR, network detection, DFIR, and threat-hunting workloads. |
| Management Host | 16 GB laptop | Dedicated administrative system used to manage Proxmox, OPNsense, SW-Lab-01, servers, and other infrastructure without being repurposed as a lab target. |
| Redteam Host | 16 GB laptop / Kali Linux bare metal | Dedicated offensive-security system for scanning, enumeration, attack simulation, exploitation exercises, and defensive-control validation. |

## Network Placement

| System | Network placement |
|---|---|
| Management Host | VLAN 10 — MANAGEMENT |
| `opnsense-fw` | WAN upstream to Archer AXE75; Layer-3 gateways for VLANs 10/20/30/40/50/60 |
| `SW-Lab-01` | Management on VLAN 10; Layer-2 transport for VLANs 10/20/30/40/50/60 |
| Enterprise Virtualization Host | Proxmox management on VLAN 10; guests on VLANs 20, 30, and 50 |
| Security Virtualization Host | Proxmox management on VLAN 10; security workloads on VLAN 40; passive sensor path planned separately |
| Redteam Host | VLAN 60 — REDTEAM |

## Enterprise Virtualization Host

**Platform:** 32 GB desktop running Proxmox VE.  
**Role:** Provide the virtualized enterprise systems required for Active Directory, Windows endpoint management, Linux administration, web services, file services, and security testing.

| VM | Recommended RAM | Primary function | VLAN | Purpose in the home lab |
|---|---:|---|---|---|
| DC01 — Windows Server | 4 GB | Active Directory Domain Services + DNS | VLAN 50 — SERVERS | Provides the lab domain, centralized authentication, directory services, domain-joined identity, Group Policy testing, and AD-integrated DNS. |
| Ubuntu Server | 4 GB | Linux server | VLAN 50 — SERVERS | General-purpose Linux server for administration, SSH, services, hardening, logging, automation, and security exercises. |
| Web Server | 2 GB | DMZ web application/server | VLAN 30 — DMZ | Hosts an isolated web service for administration, hardening, logging, monitoring, and controlled attack/defense exercises. |
| Windows 11 Workstation | 8 GB | Domain-joined user endpoint | VLAN 20 — USERS | Simulates a normal enterprise endpoint for Active Directory, Group Policy, endpoint logging, Sysmon, Wazuh agent deployment, and red-team/blue-team exercises. |
| File Server | 4 GB | Centralized file services | VLAN 50 — SERVERS | Provides SMB/file shares, NTFS permissions, user/group access testing, audit logging, file-security scenarios, and centralized storage exercises. |

### Enterprise Host Memory Plan

| Allocation | RAM |
|---|---:|
| Guest VM allocation | 22 GB |
| Reserved for Proxmox host | 4 GB |
| Unallocated headroom | 6 GB |
| Physical RAM | 32 GB |

The 6 GB of remaining capacity is intentionally left available for hypervisor overhead, caching, temporary lab activity, short-lived test VMs, and modest growth. The design does not require every VM to run at maximum utilization simultaneously.

## Security Virtualization Host

**Platform:** 32 GB Ryzen laptop running Proxmox VE.  
**Role:** Dedicated blue-team platform for centralized telemetry, SIEM/XDR, network intrusion detection, protocol analysis, threat hunting, and digital forensics.

| Security VM | Recommended RAM | Core tools / role | VLAN | Purpose in the home lab |
|---|---:|---|---|---|
| Wazuh SIEM/XDR VM | 8 GB | Wazuh manager, indexer, dashboard | VLAN 40 — SECOPS | Centralizes endpoint/server telemetry, correlates alerts, performs file-integrity monitoring, security configuration assessment, vulnerability visibility, and investigation. |
| Network Security Monitoring VM | 8 GB | Suricata + Zeek | VLAN 40 — SECOPS + passive sensor path | Receives mirrored network traffic, performs signature-based IDS inspection with Suricata, and creates network/protocol metadata with Zeek. |
| DFIR / Threat Hunting VM | 4 GB | Velociraptor / investigation tools | VLAN 40 — SECOPS | Provides endpoint visibility, artifact collection, live response, triage, threat hunting, and digital-forensics workflows without placing those tools on the Management Host. |

### Security Host Memory Plan

| Allocation | RAM |
|---|---:|
| Guest VM allocation | 20 GB |
| Reserved for Proxmox host | 4 GB |
| Unallocated headroom | 8 GB |
| Physical RAM | 32 GB |

## Security Tooling Notes

Wazuh is the baseline central SIEM/XDR platform. Suricata and Zeek share the Network Security Monitoring VM, while Velociraptor remains separate for DFIR and threat hunting. Splunk or Security Onion may be introduced later as rotational training workloads rather than always-on baseline VMs.

## Intended Telemetry Flow

Enterprise servers and endpoints send host telemetry to Wazuh. `SW-Lab-01` can mirror selected network traffic to the Network Security Monitoring VM for Suricata and Zeek analysis. The DFIR VM is used when deeper endpoint collection, triage, or threat hunting is required. The Management Host remains separate from these workloads and is used to administer the environment.

## Source Basis

- *Home Lab Hardware Inventory* — authoritative physical hardware and VM memory plan.
- *Home Lab Network Topology Overview* — network placement and virtualization VLAN mapping.
- *OPNsense Home Lab Configuration, Phases 3–12* — current firewall/switch interface baseline and management addressing.
