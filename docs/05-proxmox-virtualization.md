# Proxmox Virtualization

## Documentation Status

The source material defines the **virtualization architecture, workload placement, and memory plan**, but it does not document exact Proxmox bridge names, Linux interface names, storage pools, VM IDs, CPU allocations, or live firewall configuration. Those implementation details should be added only after they are captured from the running Proxmox hosts.

## Virtualization Design

Two separate 32 GB Proxmox VE hosts divide enterprise workloads from security-monitoring/DFIR workloads.

| Host | Platform | Primary role | Management placement | Guest/workload placement |
|---|---|---|---|---|
| Enterprise Virtualization Host | 32 GB desktop / Proxmox VE | Small-enterprise servers and Windows endpoint simulation | VLAN 10 — MANAGEMENT | VLAN 20 USERS, VLAN 30 DMZ, VLAN 50 SERVERS |
| Security Virtualization Host | 32 GB Ryzen laptop / Proxmox VE | SIEM/XDR, network security monitoring, DFIR, and threat hunting | VLAN 10 — MANAGEMENT | VLAN 40 — SECOPS plus a separate passive monitoring path when implemented |

## Enterprise Virtualization Host

### Baseline Workloads

| VM | RAM | VLAN | Role |
|---|---:|---|---|
| DC01 — Windows Server | 4 GB | 50 SERVERS | Active Directory Domain Services + DNS |
| Ubuntu Server | 4 GB | 50 SERVERS | General-purpose Linux server |
| Web Server | 2 GB | 30 DMZ | Isolated web application/server |
| Windows 11 Workstation | 8 GB | 20 USERS | Domain-joined user endpoint |
| File Server | 4 GB | 50 SERVERS | Centralized file services |

### Memory Plan

| Allocation | RAM |
|---|---:|
| Guest VM allocation | 22 GB |
| Reserved for Proxmox host | 4 GB |
| Unallocated headroom | 6 GB |
| Physical RAM | 32 GB |

The 6 GB headroom is intentionally retained for Proxmox overhead, caching, temporary lab activity, short-lived test VMs, and modest growth.

## Security Virtualization Host

### Baseline Workloads

| VM | RAM | VLAN / interface role | Function |
|---|---:|---|---|
| Wazuh SIEM/XDR VM | 8 GB | VLAN 40 — SECOPS | Wazuh manager, indexer, dashboard |
| Network Security Monitoring VM | 8 GB | VLAN 40 — SECOPS + passive monitoring interface | Suricata + Zeek |
| DFIR / Threat Hunting VM | 4 GB | VLAN 40 — SECOPS | Velociraptor / investigation tools |

### Memory Plan

| Allocation | RAM |
|---|---:|
| Guest VM allocation | 20 GB |
| Reserved for Proxmox host | 4 GB |
| Unallocated headroom | 8 GB |
| Physical RAM | 32 GB |

## Network Design Requirements

### Management Plane

Both Proxmox hosts are administered from VLAN 10 MANAGEMENT. The management plane is intentionally separated from normal user, DMZ, server, security-workload, and red-team traffic.

### Enterprise Guest Networks

The Enterprise host must be capable of presenting VLANs 20, 30, and 50 to the appropriate VMs while retaining host management on VLAN 10.

### Security Guest Networks

The Security host uses VLAN 40 for Wazuh, Suricata/Zeek management, and Velociraptor/DFIR.

### Passive Sensor Path

The Network Security Monitoring VM is designed to receive mirrored switch traffic over a separate sensor path. This passive NIC is conceptually distinct from the normal VLAN 40 management interface. The source design does not yet assign the final Aruba mirror destination port, so the repository must not invent one.

## Optional / Rotational Workloads

Splunk and Security Onion are not part of the always-on baseline. They may be introduced later as rotational training workloads so the core Wazuh/NSM/DFIR design retains the required RAM headroom.

## Implementation Details Still Requiring Live Evidence

The following should remain undocumented or explicitly marked pending until verified from the hosts:

- Proxmox node hostnames
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

## Evidence Targets

When the live configuration is ready, capture:

- Proxmox node summary for each host
- Network/bridge configuration showing VLAN-aware design
- Enterprise VM inventory
- Security VM inventory
- Per-VM network/VLAN assignment
- Host management placement on VLAN 10
- Security sensor's distinct monitoring NIC after the mirror path is implemented

## Source Basis

- *Home Lab Hardware Inventory* — virtualization host roles, VM allocations, and RAM plans.
- *Home Lab Network Topology Overview* — VLAN placement and passive security-sensor path.
- *OPNsense Home Lab Configuration, Phases 3–12* — VLAN gateway design and current switch trunk baseline.
