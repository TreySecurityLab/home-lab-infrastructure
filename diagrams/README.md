# Diagrams

This directory contains the portfolio-facing architecture diagrams for the home lab.

- [`home-lab-architecture.png`](home-lab-architecture.png) — physical/logical overview from the ISP handoff through OPNsense, SW-Lab-01, management endpoints, Proxmox hosts, and planned workload placement.
- [`vlan-topology.png`](vlan-topology.png) — authoritative VLAN IDs, subnets, gateways, named systems, security purpose, and virtualization placement.
- [`security-telemetry-flow.png`](security-telemetry-flow.png) — planned/conditional telemetry paths for Wazuh, Suricata + Zeek, and Velociraptor/DFIR, with unverified integrations clearly marked as pending deployment or validation.

## Authoritative Source Basis

- `Home-Lab-Network-Topology-final-2026-08-20.pdf`
- `Home-Lab-Hardware-Inventory-final-2026-08-20.pdf`
- `Authoritative-VLAN-Designations-final-2026-08-20.txt`
- `OPNsense-Phases-3-12-Home-Lab-Configuration-final-2026-08-20.pdf`

The diagrams represent the approved design baseline. A planned VM, guest VLAN, security agent, passive sensor path, or SPAN/mirror integration must not be treated as operational until deployment and verification are complete.
