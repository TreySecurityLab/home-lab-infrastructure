# Home Lab Infrastructure

Enterprise-style home lab designed for network segmentation, infrastructure administration, virtualization, defensive monitoring, offensive-security testing, and validation of security controls.

> **Documentation status:** This repository records the current source-supported architecture and clearly marks planned or pending implementation items. Temporary recovery settings are not presented as permanent topology.

## Architecture Summary

**Internet → Archer AXE75 → OPNsense firewall → SW-Lab-01 (Aruba J9774A) → segmented VLANs and Proxmox workloads**

OPNsense is the Layer-3 security boundary for Internet access and inter-VLAN policy. SW-Lab-01 provides Layer-2 VLAN segmentation and 802.1Q transport. Selected switch traffic can eventually be mirrored to a passive Suricata + Zeek sensor interface for network-security monitoring.

## Core Infrastructure

| Component | Platform | Role |
|---|---|---|
| Archer AXE75 | Wi-Fi router | Upstream home network and Internet connectivity |
| `opnsense-fw` | Bare-metal OPNsense / 8 GB RAM | Perimeter firewall, NAT, VLAN routing, DHCP/DNS design, and policy enforcement |
| `SW-Lab-01` | Aruba J9774A managed switch | Layer-2 VLAN segmentation, 802.1Q tagging/trunking, access ports, and traffic mirroring |
| Enterprise Virtualization Host | 32 GB desktop / Proxmox VE | Enterprise server and workstation VMs |
| Security Virtualization Host | 32 GB Ryzen laptop / Proxmox VE | Wazuh SIEM/XDR, Suricata/Zeek NSM, and Velociraptor/DFIR workloads |
| Management Host | 16 GB laptop | Dedicated infrastructure administration |
| Redteam Host | 16 GB laptop / Kali Linux bare metal | Offensive-security testing and security-control validation |

## VLAN Design

| VLAN | Name | Network | Gateway | Primary purpose |
|---:|---|---|---|---|
| 10 | MANAGEMENT | 10.10.10.0/24 | 10.10.10.1 | Administrative control plane |
| 20 | USERS | 10.10.20.0/24 | 10.10.20.1 | Simulated enterprise user endpoints |
| 30 | DMZ | 10.10.30.0/24 | 10.10.30.1 | Isolated web workload and controlled attack/defense testing |
| 40 | SECOPS | 10.10.40.0/24 | 10.10.40.1 | Monitoring, detection, SIEM/XDR, and DFIR |
| 50 | SERVERS | 10.10.50.0/24 | 10.10.50.1 | Active Directory, Linux, and file services |
| 60 | REDTEAM | 10.10.60.0/24 | 10.10.60.1 | Offensive-security and attack simulation |

## Current Switching Baseline

- `SW-Lab-01` port 1 — OPNsense trunk; Tagged VLANs 10/20/30/40/50/60
- `SW-Lab-01` port 8 — MANAGEMENT PC access; Untagged VLAN 10
- `SW-Lab-01` management — 10.10.10.2/24; gateway 10.10.10.1
- All other switch ports — TBD; no permanent role should be inferred

## OPNsense Policy Baseline

- MANAGEMENT and SECOPS can reach lab networks and administer the OPNsense WebUI over HTTPS.
- USERS, DMZ, SERVERS, and REDTEAM are blocked from initiating traffic to other lab VLANs.
- All zones are permitted DNS to OPNsense on TCP/UDP 53.
- Restricted zones are blocked from other management access to OPNsense.
- Outbound/Internet access is allowed according to the documented zone policy.
- `LAB_NETWORKS` represents all six internal VLAN networks for readable firewall rules.
- Dnsmasq provides DHCP on USERS, SECOPS, and REDTEAM with Listen Port 0; Unbound remains the DNS resolver on port 53.

## Virtualization Placement

The Enterprise Virtualization Host uses VLAN 10 for Proxmox management and carries guest workloads on VLANs 20, 30, and 50. The Security Virtualization Host uses VLAN 10 for Proxmox management and VLAN 40 for the security stack. The Redteam Host is isolated on VLAN 60.

## Security Telemetry

- Windows 11, DC01, Ubuntu Server, File Server, and Web Server provide host telemetry to Wazuh.
- Selected USERS, SERVERS, DMZ, and REDTEAM traffic can be mirrored from SW-Lab-01 to a passive Suricata + Zeek sensor interface once the mirror destination is finalized.
- Velociraptor supports endpoint triage, collection, and threat-hunting workflows.
- The Management Host remains separate from security workloads and investigation targets.

## Documentation

- [Documentation Index](docs/README.md)
- [Hardware Inventory](docs/01-hardware-inventory.md)
- [Network Architecture](docs/02-network-architecture.md)
- [Aruba Switching](docs/03-aruba-switching.md)
- [OPNsense Firewall](docs/04-opnsense-firewall.md)
- [Proxmox Virtualization](docs/05-proxmox-virtualization.md)
- [Virtual Machines](docs/06-virtual-machines.md)
- [Security Controls](docs/07-security-controls.md)
- [Validation Testing](docs/08-validation-testing.md)

## Evidence Model

Each implemented control should be documented as **Design → Configuration → Security Purpose → Evidence → Validation**. Screenshots and sanitized configuration extracts should demonstrate the resulting state rather than merely showing commands being entered.

## Security / Sanitization

Do not commit passwords, private keys, API tokens, VPN secrets, recovery codes, SNMP community strings, public WAN details, serial numbers, or other sensitive identifiers. Sanitized configurations belong under `configs/sanitized/`.
