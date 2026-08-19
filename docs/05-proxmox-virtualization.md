# Proxmox Virtualization

## Documentation Status

The source material defines the virtualization architecture, workload placement, host management addresses, and baseline memory plans. Exact Proxmox bridge names, Linux interface names, storage pools, VM IDs, CPU allocations, and live firewall configuration remain pending until captured from the running hosts.

## Virtualization Design

Two separate 32 GB Proxmox VE hosts divide enterprise workloads from security-monitoring/DFIR workloads.

| Host | Address | Platform | Primary role | Management placement | Guest/workload placement |
|---|---|---|---|---|---|
| `ENTHOST-01` | 10.10.10.3 | 32 GB desktop / Proxmox VE | Small-enterprise servers and Windows endpoint simulation | VLAN 10 — MANAGEMENT | VLAN 20 USERS, VLAN 30 DMZ, VLAN 50 SERVERS |
| `SECHOST-01` | 10.10.10.4 | 32 GB Ryzen laptop / Proxmox VE | SIEM/XDR, network security monitoring, DFIR, and threat hunting | VLAN 10 — MANAGEMENT | VLAN 40 — SECOPS plus a separate passive monitoring path when implemented |

## `ENTHOST-01` — Enterprise Virtualization Host

### Baseline Workloads

| VM | Address | RAM | VLAN | Role |
|---|---|---:|---|---|
| `DC-01` — Windows Server | 10.10.50.10 | 4 GB | 50 SERVERS | Active Directory Domain Services + DNS |
| `LINUX-01` — Ubuntu Server | 10.10.50.40 | 4 GB | 50 SERVERS | General-purpose Linux server |
| `WEB-01` — Web Server | 10.10.30.10 | 2 GB | 30 DMZ | Isolated web application/server |
| `WIN11-01` — Windows 11 Workstation | 10.10.20.10 | 8 GB | 20 USERS | Domain-joined user endpoint |
| `FILE-01` — File Server | 10.10.50.30 | 4 GB | 50 SERVERS | Centralized file services |
| `DC-02` — Windows Server | 10.10.50.20 | 3–4 GB | 50 SERVERS | Planned/rotational second domain controller + DNS; deploy after `DC-01` is stable |

### Memory Plan

The normal always-on baseline for the five primary enterprise VMs remains:

| Allocation | RAM |
|---|---:|
| Always-on baseline guest VM allocation | 22 GB |
| Reserved for Proxmox host | 4 GB |
| Unallocated baseline headroom | 6 GB |
| Physical RAM | 32 GB |
| `DC-02` rotational allocation | Approximately 3–4 GB additional only while active |

`DC-02` is intentionally **not** included in the normal 22 GB guest total. When it is powered on for second-domain-controller exercises, its approximately 3–4 GB comes from available headroom or by rotating another workload off. Do not treat `DC-02` as continuously running unless the architecture is later revised and re-verified.

## `SECHOST-01` — Security Virtualization Host

### Baseline Workloads

| VM | RAM | VLAN / interface role | Function |
|---|---:|---|---|
| Wazuh SIEM/XDR VM | 8 GB | VLAN 40 — SECOPS | Wazuh manager, indexer, dashboard |
| Network Security Monitoring VM | 8 GB | VLAN 40 — SECOPS + passive monitoring interface | Suricata + Zeek |
| DFIR / Threat Hunting VM | 4 GB | VLAN 40 — SECOPS | Velociraptor / investigation tools |

The final authoritative VLAN designation source does not yet assign static IP addresses to these VLAN 40 workloads, so none are invented here.

### Memory Plan

| Allocation | RAM |
|---|---:|
| Guest VM allocation | 20 GB |
| Reserved for Proxmox host | 4 GB |
| Unallocated headroom | 8 GB |
| Physical RAM | 32 GB |

## Network Design Requirements

### Management Plane

Both Proxmox hosts are administered from VLAN 10 MANAGEMENT:

- `ENTHOST-01` — 10.10.10.3
- `SECHOST-01` — 10.10.10.4
- Administrative access originates from `MGMT-01` — 10.10.10.6

### Enterprise Guest Networks

`ENTHOST-01` must be capable of presenting VLANs 20, 30, and 50 to the appropriate VMs while retaining host management on VLAN 10.

### Security Guest Networks

`SECHOST-01` uses VLAN 40 for Wazuh, Suricata/Zeek management, and Velociraptor/DFIR.

### Passive Sensor Path

The Network Security Monitoring VM is designed to receive mirrored switch traffic over a separate sensor path. This passive NIC is conceptually distinct from the normal VLAN 40 management interface. The final Aruba mirror destination port is not assigned in the current source and must not be invented.

## Optional / Rotational Workloads

`DC-02` is a planned/rotational Enterprise workload. Splunk and Security Onion are also not part of their hosts' always-on baselines and may be introduced later as rotational training workloads so required RAM headroom is preserved.

## Implementation Details Still Requiring Live Evidence

- Exact Proxmox node/FQDN configuration beyond the authoritative inventory labels
- Physical NIC/interface names
- Linux bridge names
- VLAN-aware bridge configuration
- Storage pools and datastore layout
- VM IDs
- CPU/vCPU assignments
- Boot order and autostart policy
- Proxmox firewall rules
- Backup jobs and destinations
- Exact passive sensor NIC / mirror-destination port
- `DC-02` vCPU, storage, and final deployment evidence

## Evidence Targets

When live configuration is ready, capture node summaries, network/bridge configuration, VM inventories, per-VM VLAN assignments, host management placement, storage configuration, and the separate security-sensor NIC after the mirror path is implemented.

## Verified-State Rule

Planned VMs, VLAN placement, and sensor paths described here are design targets only. They become operational facts only after installation, configuration, testing, and verification evidence are complete.

## Source Basis

- *Home Lab Hardware Inventory — UPDATED* — virtualization host roles, VM allocations, and RAM plans.
- *Authoritative VLAN Designations — final* — Proxmox host labels/addresses and enterprise workload hostname/IP assignments.
- *Home Lab Network Topology Overview — UPDATED* — VLAN placement and passive security-sensor path.
- *OPNsense Home Lab Configuration — UPDATED* — VLAN gateway design and current switch trunk baseline.
