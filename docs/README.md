# Documentation Index

This directory contains the detailed infrastructure and security documentation for the home lab. The numbered files follow the logical build and verification order: physical inventory, network architecture, switching, firewalling, virtualization, workloads, security controls, and validation.

| File | Purpose | Source-supported status |
|---|---|---|
| [01-hardware-inventory.md](01-hardware-inventory.md) | Physical systems, roles, VM allocations, memory plans | Populated from authoritative hardware inventory |
| [02-network-architecture.md](02-network-architecture.md) | Physical path, VLANs, gateways, switch ports, virtualization placement, telemetry | Populated from topology and OPNsense sources |
| [03-aruba-switching.md](03-aruba-switching.md) | J9774A role, final port baseline, VLAN membership, verification commands | Populated from current OPNsense port baseline + Aruba AOS-S guide |
| [04-opnsense-firewall.md](04-opnsense-firewall.md) | Interfaces, VLAN devices, gateways, DHCP/DNS, aliases, firewall policy, recovery history | Populated from Phases 3–12 source document |
| [05-proxmox-virtualization.md](05-proxmox-virtualization.md) | Host roles, VLAN placement, memory plan, implementation items still needing live evidence | Populated to source-supported level |
| [06-virtual-machines.md](06-virtual-machines.md) | Enterprise and security VM roles, RAM, VLANs, telemetry relationships | Populated from hardware/topology sources |
| [07-security-controls.md](07-security-controls.md) | Segmentation, management separation, firewall control patterns, monitoring/DFIR controls | Populated from current sources; unsupported controls marked future evidence |
| [08-validation-testing.md](08-validation-testing.md) | Acceptance tests, policy matrix, evidence plan, pending validation | Populated from OPNsense acceptance-test source |

## Documentation Rule

Only source-supported or live-verified values are documented as current. Planned items remain explicitly marked planned/pending, and temporary recovery settings are retained only as historical notes rather than presented as permanent architecture.
