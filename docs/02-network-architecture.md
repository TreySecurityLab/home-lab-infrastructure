# Network Architecture

## Architecture Summary

The home lab is designed around a routed security boundary with Layer-2 VLAN transport behind it:

**Internet / ISP → `opnsense-fw` → `SW-Lab-01` (Aruba J9774A) → segmented VLANs, management systems, Proxmox hosts, and lab workloads**

`opnsense-fw` is directly connected to the ISP and is the Layer-3 gateway and policy-enforcement point for all six lab VLANs. `SW-Lab-01` is Layer-2 only and transports 802.1Q-tagged traffic between OPNsense and VLAN-aware hosts.

No separate consumer router is part of the authoritative home-lab topology.

## Authoritative VLAN Topology

| VLAN | Name | Network | OPNsense gateway | Primary systems / role | Security purpose |
|---:|---|---|---|---|---|
| 10 | MANAGEMENT | 10.10.10.0/24 | 10.10.10.1 | `MGMT-01`, `MGMT-BACKUP`, OPNsense management, `SW-Lab-01`, `ENTHOST-01`, `SECHOST-01` | Administrative control plane |
| 20 | USERS | 10.10.20.0/24 | 10.10.20.1 | `WIN11-01` | Simulated enterprise user / endpoint network |
| 30 | DMZ | 10.10.30.0/24 | 10.10.30.1 | `WEB-01` | Isolated web-server and controlled attack/defense zone |
| 40 | SECOPS | 10.10.40.0/24 | 10.10.40.1 | Wazuh, Suricata/Zeek management, Velociraptor / DFIR | Blue-team monitoring, detection, and investigation |
| 50 | SERVERS | 10.10.50.0/24 | 10.10.50.1 | `DC-01`, `DC-02` (planned/rotational), `FILE-01`, `LINUX-01` | Internal enterprise infrastructure |
| 60 | REDTEAM | 10.10.60.0/24 | 10.10.60.1 | `REDTEAM-01` | Offensive-security and attack-simulation network |

`DC-02` is intentionally part of VLAN 50 as a planned/rotational second domain controller at 10.10.50.20. Its recommended RAM is approximately 3–4 GB while active, and it is not part of the normal always-on 22 GB Enterprise VM allocation. Remaining implementation details such as vCPU and storage should be recorded only after deployment planning is finalized.

## Authoritative Host / IP Assignments

| VLAN | Host | Address |
|---:|---|---|
| 10 | `OPNsense-FW` (configured hostname `opnsense-fw`) | 10.10.10.1 |
| 10 | `SW-LAB-01` / `SW-Lab-01` | 10.10.10.2 |
| 10 | `ENTHOST-01` | 10.10.10.3 |
| 10 | `SECHOST-01` | 10.10.10.4 |
| 10 | `MGMT-01` | 10.10.10.5 |
| 10 | `MGMT-BACKUP` | 10.10.10.6 when locally attached |
| 20 | `WIN11-01` | 10.10.20.10 |
| 30 | `WEB-01` | 10.10.30.10 |
| 50 | `DC-01` | 10.10.50.10 |
| 50 | `DC-02` | 10.10.50.20 |
| 50 | `FILE-01` | 10.10.50.30 |
| 50 | `LINUX-01` | 10.10.50.40 |
| 60 | `REDTEAM-01` | 10.10.60.10 |

No authoritative static host assignments are currently listed for VLAN 40 SECOPS workloads in the final designation source.

## Physical Interface and Switch-Port Baseline

| Component | Interface / port | Current role | Tagging / addressing |
|---|---|---|---|
| Internet / ISP | Service handoff | Direct upstream for OPNsense WAN | ISP-supplied addressing; do not hard-code public/ISP-assigned details |
| `opnsense-fw` | `re0` | WAN | Direct ISP-facing WAN |
| `opnsense-fw` | `ue0` (validated runtime) | LAN/VLAN parent toward `SW-Lab-01` | Carries VLAN tags 10/20/30/40/50/60 |
| `SW-Lab-01` | Port 1 | OPNsense trunk | Tagged VLANs 10/20/30/40/50/60; no final untagged lab VLAN documented |
| `SW-Lab-01` | Port 6 | `MGMT-BACKUP` access | Untagged VLAN 10 |
| `SW-Lab-01` | Port 7 | `ENTHOST-01` physical attachment | Final VLAN-aware Proxmox trunk/tagging details remain pending live validation |
| `SW-Lab-01` | Port 8 | `MGMT-01` access | Untagged VLAN 10 |
| `SW-Lab-01` | Ports 2–5, 9+ | TBD | No permanent lab role documented; do not infer |
| `SW-Lab-01` | Management SVI | Switch management | 10.10.10.2/24; default gateway 10.10.10.1 |

### Historical Recovery Path

Port 7 and 192.168.99.0/24 were used temporarily while VLAN 10 was repaired. That temporary VLAN 1/recovery function is historical only. Port 7 has since been assigned as the physical attachment for `ENTHOST-01`; its final Proxmox VLAN-aware guest trunk configuration remains pending verification.

## Virtualization VLAN Placement

### `ENTHOST-01` — Enterprise Virtualization Host

- Physical switch attachment — SW-Lab-01 port 7
- VLAN 10 — Proxmox management, 10.10.10.3
- VLAN 20 — `WIN11-01`, 10.10.20.10
- VLAN 30 — `WEB-01`, 10.10.30.10
- VLAN 50 — `DC-01`, `DC-02` (planned/rotational), `FILE-01`, `LINUX-01`

`DC-02` is deployed only after `DC-01` is stable. Its approximately 3–4 GB RAM is rotational and remains outside the normal always-on 22 GB Enterprise guest allocation.

### `SECHOST-01` — Security Virtualization Host

- VLAN 10 — Proxmox management, 10.10.10.4
- VLAN 40 — Wazuh SIEM/XDR
- VLAN 40 — Suricata + Zeek management
- VLAN 40 — Velociraptor / DFIR
- Separate passive sensor interface — receives mirrored traffic from `SW-Lab-01` when the SPAN/mirror destination is implemented

### `REDTEAM-01` — Redteam Host

- VLAN 60 — bare-metal Kali Linux, 10.10.60.10

## DHCP and Addressing Model

Dynamic addressing is used only where endpoint churn is expected. Infrastructure and service networks use static/reserved addressing.

| VLAN | DHCP | Addressing policy |
|---:|---|---|
| 10 MANAGEMENT | No | Static/reserved infrastructure addresses |
| 20 USERS | Yes | 10.10.20.100–10.10.20.199, while `WIN11-01` has an authoritative static assignment of 10.10.20.10 |
| 30 DMZ | No | Static service addresses |
| 40 SECOPS | Yes | 10.10.40.100–10.10.40.199 |
| 50 SERVERS | No | Static server addresses |
| 60 REDTEAM | Yes | 10.10.60.100–10.10.60.199, while `REDTEAM-01` has an authoritative static assignment of 10.10.60.10 |

## Security Telemetry Flow

### Host telemetry → Wazuh

The topology defines Windows 11, `DC-01`, `LINUX-01`, `FILE-01`, and `WEB-01` as intended host-telemetry sources for Wazuh on VLAN 40 after their agents are deployed and verified. `DC-02` joins that telemetry flow only while the rotational VM is deployed and after its Wazuh agent is installed and verified.

### Mirrored network traffic → Suricata + Zeek

Selected traffic from USERS, SERVERS, DMZ, and REDTEAM can be mirrored by `SW-Lab-01` to a passive sensor NIC. Suricata performs IDS inspection while Zeek produces network/protocol metadata for investigation.

### Investigation / collection → Velociraptor

Suspicious endpoints can be triaged and collected through the Velociraptor/DFIR workload on VLAN 40.

### Management separation

`MGMT-01` remains the primary dedicated management system, while `MGMT-BACKUP` provides a secondary management path when required. VLAN 10 is the administrative control plane and is intentionally distinct from USERS, DMZ, SECOPS workload traffic, SERVERS, and REDTEAM.

## Verified-State Rule

This document is the approved design baseline. A VM, VLAN placement, security agent, passive sensor interface, or mirror path must not be presented as operational until it has been installed, configured, tested, and verified.

## Source Basis

- `Authoritative-VLAN-Designations-final-2026-08-20.txt` — VLAN IDs, names, /24 networks, and host/IP assignments.
- `Home-Lab-Network-Topology-final-2026-08-20.pdf` — direct ISP path, workload placement, virtualization trunks, and telemetry flow.
- `OPNsense-Phases-3-12-Home-Lab-Configuration-final-2026-08-20.pdf` — direct ISP WAN, gateway, DHCP, interface, and switch-port baseline.
- `Home-Lab-Hardware-Inventory-final-2026-08-20.pdf` — physical/virtual host roles and memory plans.
- Supplemental verified management-network evidence — verified port 6 VLAN 10 assignment and backup-management addressing.
- Verified SW-Lab-01 live configuration captured August 20, 2026 — current port 1/6/7/8 assignments.
