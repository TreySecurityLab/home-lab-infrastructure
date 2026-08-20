# Home Lab Infrastructure

Enterprise-style cybersecurity home lab documenting network segmentation, OPNsense firewalling, Aruba switching, Proxmox virtualization, security monitoring, and validation.

> **Documentation status:** This repository records the current source-supported architecture and clearly marks planned or pending implementation items. Temporary recovery settings are not presented as permanent topology.

## Architecture Summary

**Internet / ISP → OPNsense firewall → SW-Lab-01 (Aruba J9774A) → segmented VLANs, Proxmox hosts, endpoints, and security workloads**

OPNsense is directly connected to the ISP and is the lab edge firewall/router. It is the Layer-3 gateway and policy-enforcement point for the six lab VLANs. SW-Lab-01 provides Layer-2 VLAN segmentation and 802.1Q transport.

## Core Infrastructure

| Component | Platform | Role |
|---|---|---|
| `opnsense-fw` | Bare-metal OPNsense / 8 GB RAM | ISP-facing edge firewall/router, NAT, VLAN routing, DHCP/DNS design, and policy enforcement |
| `SW-Lab-01` | Aruba J9774A managed switch | Layer-2 VLAN segmentation, 802.1Q tagging/trunking, access ports, and traffic mirroring |
| `ENTHOST-01` | 32 GB desktop / Proxmox VE | Enterprise server and workstation virtualization host |
| `SECHOST-01` | 32 GB Ryzen laptop / Proxmox VE | Wazuh SIEM/XDR, Suricata/Zeek NSM, and Velociraptor/DFIR virtualization host |
| `MGMT-01` | 16 GB laptop | Dedicated infrastructure administration system |
| `MGMT-BACKUP` | Laptop | Backup/remote management system; local VLAN 10 address 10.10.10.10/24 when physically connected |
| `READTEAM-01` | 16 GB laptop / Kali Linux bare metal | Offensive-security testing and defensive-control validation |

No separate consumer router is part of the home-lab infrastructure. OPNsense is the first lab routing/security device on the ISP-facing connection.

## VLAN Design

| VLAN | Name | Network | Gateway | Primary purpose |
|---:|---|---|---|---|
| 10 | MANAGEMENT | 10.10.10.0/24 | 10.10.10.1 | Administrative control plane |
| 20 | USERS | 10.10.20.0/24 | 10.10.20.1 | Simulated enterprise endpoints |
| 30 | DMZ | 10.10.30.0/24 | 10.10.30.1 | Isolated web/application services |
| 40 | SECOPS | 10.10.40.0/24 | 10.10.40.1 | Monitoring, detection, SIEM/XDR, and DFIR |
| 50 | SERVERS | 10.10.50.0/24 | 10.10.50.1 | Active Directory, Linux, and file services |
| 60 | REDTEAM | 10.10.60.0/24 | 10.10.60.1 | Offensive-security and attack simulation |

## Authoritative Host / IP Assignments

| VLAN | Host | Address | Status / role |
|---:|---|---|---|
| 10 | `opnsense-fw` / inventory label `OPNsense-FW` | 10.10.10.1 | OPNsense MANAGEMENT gateway |
| 10 | `SW-Lab-01` / inventory label `SW-LAB-01` | 10.10.10.2 | Switch management |
| 10 | `ENTHOST-01` | 10.10.10.3 | Enterprise Proxmox management |
| 10 | `SECHOST-01` | 10.10.10.4 | Security Proxmox management |
| 10 | `MGMT-01` | 10.10.10.6 | Management Host |
| 10 | `MGMT-BACKUP` | 10.10.10.10 | Backup management laptop when locally attached to VLAN 10 |
| 20 | `WIN11-01` | 10.10.20.10 | Windows 11 workstation |
| 30 | `WEB-01` | 10.10.30.10 | DMZ web server |
| 50 | `DC-01` | 10.10.50.10 | Primary domain-controller workload |
| 50 | `DC-02` | 10.10.50.20 | Planned/rotational second domain controller; approximately 3–4 GB RAM when active |
| 50 | `FILE-01` | 10.10.50.30 | File server |
| 50 | `LINUX-01` | 10.10.50.40 | Ubuntu/Linux server |
| 60 | `READTEAM-01` | 10.10.60.10 | Bare-metal red-team host |

`DC-02` is intentionally part of the planned home lab as a rotational second domain controller at 10.10.50.20 on VLAN 50 SERVERS. Its recommended RAM is approximately 3–4 GB, and it is not part of the normal always-on 22 GB Enterprise VM allocation unless the memory plan is later revised and re-verified.

## Current Switching Baseline

- `SW-Lab-01` port 1 — OPNsense trunk; tagged VLANs 10/20/30/40/50/60
- `SW-Lab-01` port 6 — `MGMT-BACKUP`; untagged VLAN 10 MANAGEMENT access
- `SW-Lab-01` port 7 — `ENTHOST-01` physical attachment; final Proxmox VLAN-aware guest-trunk details remain pending live validation
- `SW-Lab-01` port 8 — `MGMT-01`; untagged VLAN 10 MANAGEMENT access
- `SW-Lab-01` management — 10.10.10.2/24; gateway 10.10.10.1
- Ports 2–5 and 9+ — TBD; no permanent role should be inferred

## Completed SW-Lab-01 Management and Security Baseline — August 20, 2026

- Aruba J9774A management address and gateway verified at 10.10.10.2/24 and 10.10.10.1.
- Firmware upgraded through the validated TFTP workflow to `YA.16.10.0010` and the switch returned normally after reboot.
- VLAN and management configuration were saved and verified after reboot.
- An off-switch configuration backup was exported for recovery/reference.
- Time synchronization was configured and verified.
- SNMPv3 was enabled and a dedicated authenticated monitoring user was validated from the management network.
- Legacy SNMPv1/v2c access using the former public community was tested after hardening and failed as intended, while SNMPv3 continued to succeed.
- Final sanitized screenshot evidence is stored under `screenshots/aruba/`.

## OPNsense Policy Baseline

- WAN is directly ISP-facing; ISP-assigned/public WAN details are intentionally not hard-coded in this repository.
- MANAGEMENT and SECOPS can reach lab networks and administer the OPNsense WebUI over HTTPS.
- USERS, DMZ, SERVERS, and REDTEAM are blocked from initiating traffic to other lab VLANs.
- All zones are permitted DNS to OPNsense on TCP/UDP 53.
- Restricted zones are blocked from other management access to OPNsense.
- Outbound/Internet access is allowed according to the documented zone policy.
- `LAB_NETWORKS` represents all six internal VLAN networks for readable firewall rules.
- Dnsmasq provides DHCP on USERS, SECOPS, and REDTEAM with Listen Port 0; Unbound remains the DNS resolver on port 53.

## Virtualization Placement

`ENTHOST-01` uses VLAN 10 for Proxmox management and carries guest workloads on VLANs 20, 30, and 50. `DC-02` is a planned/rotational VLAN 50 workload deployed only after `DC-01` is stable. `SECHOST-01` uses VLAN 10 for Proxmox management and VLAN 40 for the security stack. `READTEAM-01` is assigned to VLAN 60.

## Security Telemetry

- Windows/server workloads provide host telemetry to Wazuh as they are deployed and verified.
- Selected USERS, SERVERS, DMZ, and REDTEAM traffic can be mirrored from SW-Lab-01 to a passive Suricata + Zeek sensor interface once the mirror destination is finalized.
- Velociraptor supports endpoint triage, collection, and threat-hunting workflows.
- `MGMT-01` remains separate from security workloads and investigation targets.

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

Each implemented control should be documented as **Design → Configuration → Security Purpose → Evidence → Validation**. Planned items remain explicitly marked pending until they are deployed and verified.

## Security / Sanitization

Do not commit passwords, private keys, API tokens, VPN secrets, recovery codes, SNMP community strings, public/ISP-assigned WAN details, serial numbers, complete MAC addresses, or other sensitive identifiers. Sanitized configurations belong under `configs/sanitized/`.
