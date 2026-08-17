# Network Architecture

## Architecture Summary

The home lab is designed around a routed security boundary with Layer-2 VLAN transport behind it:

**Internet → Archer AXE75 → `opnsense-fw` → `SW-Lab-01` (Aruba J9774A) → segmented VLANs, management systems, Proxmox hosts, and lab workloads**

`opnsense-fw` is the Layer-3 gateway and policy-enforcement point for all six lab VLANs. `SW-Lab-01` is Layer-2 only and transports 802.1Q-tagged traffic between OPNsense and VLAN-aware hosts.

## Authoritative VLAN Topology

| VLAN | Name | Network | OPNsense gateway | Primary systems / role | Security purpose |
|---:|---|---|---|---|---|
| 10 | MANAGEMENT | 10.10.10.0/24 | 10.10.10.1 | Management Host, OPNsense management, SW-Lab-01 management, Proxmox management | Administrative control plane |
| 20 | USERS | 10.10.20.0/24 | 10.10.20.1 | Windows 11 Workstation | Simulated enterprise user / endpoint network |
| 30 | DMZ | 10.10.30.0/24 | 10.10.30.1 | Web Server VM | Isolated web-server and controlled attack/defense zone |
| 40 | SECOPS | 10.10.40.0/24 | 10.10.40.1 | Wazuh, Suricata/Zeek management, Velociraptor / DFIR | Blue-team monitoring, detection, and investigation |
| 50 | SERVERS | 10.10.50.0/24 | 10.10.50.1 | DC01, Ubuntu Server, File Server | Internal enterprise infrastructure |
| 60 | REDTEAM | 10.10.60.0/24 | 10.10.60.1 | Kali Redteam Host | Offensive-security and attack-simulation network |

## Physical Interface and Switch-Port Baseline

| Component | Interface / port | Current role | Tagging / addressing |
|---|---|---|---|
| Archer AXE75 | LAN | Upstream for OPNsense WAN | Router: 192.168.1.1; OPNsense WAN lease is dynamic and should not be hard-coded in public documentation |
| `opnsense-fw` | `re0` | WAN | IPv4 DHCP from Archer AXE75 |
| `opnsense-fw` | `ue0` (validated runtime) | LAN/VLAN parent toward `SW-Lab-01` | Carries VLAN tags 10/20/30/40/50/60 |
| `SW-Lab-01` | Port 1 | OPNsense trunk | Tagged VLANs 10/20/30/40/50/60; no final untagged lab VLAN documented |
| `SW-Lab-01` | Port 8 | MANAGEMENT PC access | Untagged VLAN 10 |
| `SW-Lab-01` | Ports 2–7, 9+ | TBD | No permanent lab role documented; do not infer |
| `SW-Lab-01` | Management SVI | Switch management | 10.10.10.2/24; default gateway 10.10.10.1 |

### Historical Recovery Path

Port 7 and 192.168.99.0/24 were used temporarily while VLAN 10 was repaired. They are troubleshooting history only and are **not** part of the permanent topology. Port 7 remains TBD until assigned a final role.

## Virtualization VLAN Placement

### Enterprise Virtualization Host

- VLAN 10 — Proxmox management
- VLAN 20 — Windows 11 Workstation
- VLAN 30 — Web Server
- VLAN 50 — DC01, Ubuntu Server, File Server

### Security Virtualization Host

- VLAN 10 — Proxmox management
- VLAN 40 — Wazuh SIEM/XDR
- VLAN 40 — Suricata + Zeek management
- VLAN 40 — Velociraptor / DFIR
- Separate passive sensor interface — receives mirrored traffic from `SW-Lab-01` when the SPAN/mirror destination is implemented

### Redteam Host

- VLAN 60 — bare-metal Kali Linux

## DHCP and Addressing Model

Dynamic addressing is used only where endpoint churn is expected. Infrastructure and service networks use static/reserved addressing.

| VLAN | DHCP | Addressing policy |
|---:|---|---|
| 10 MANAGEMENT | No | Static/reserved infrastructure addresses |
| 20 USERS | Yes | 10.10.20.100–10.10.20.199 |
| 30 DMZ | No | Static service addresses |
| 40 SECOPS | Yes | 10.10.40.100–10.10.40.199 |
| 50 SERVERS | No | Static server addresses |
| 60 REDTEAM | Yes | 10.10.60.100–10.10.60.199 |

## Security Telemetry Flow

### Host telemetry → Wazuh

Windows 11, DC01, Ubuntu Server, File Server, and Web Server provide agent/log telemetry to Wazuh on VLAN 40. Wazuh is the centralized alerting, correlation, file-integrity monitoring, and security-configuration assessment platform.

### Mirrored network traffic → Suricata + Zeek

Selected traffic from USERS, SERVERS, DMZ, and REDTEAM can be mirrored by `SW-Lab-01` to a passive sensor NIC. Suricata performs IDS inspection while Zeek produces network/protocol metadata for investigation.

### Investigation / collection → Velociraptor

Suspicious endpoints can be triaged and collected through the Velociraptor/DFIR workload on VLAN 40.

### Management separation

The Management Host remains separate from security workloads and investigation targets. VLAN 10 is the administrative control plane and is intentionally distinct from USERS, DMZ, SECOPS workload traffic, SERVERS, and REDTEAM.

## Source Basis

- *Authoritative VLAN Designations* — VLAN IDs, names, and /24 networks.
- *Home Lab Network Topology Overview* — physical path, workload placement, virtualization trunks, and telemetry flow.
- *OPNsense Home Lab Configuration, Phases 3–12* — current gateway, DHCP, interface, and switch-port baseline.
