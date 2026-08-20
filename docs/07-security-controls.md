# Security Controls

## Control Model

The lab is built around segmentation, least exposure, dedicated administration, centralized telemetry, and explicit validation. This page documents controls supported by the current source material and verified live evidence, while separating them from controls that remain planned or require additional implementation.

## Network Segmentation

Six VLANs separate administrative, user, exposed-service, security-operations, server, and attacker-simulation functions:

- VLAN 10 — MANAGEMENT
- VLAN 20 — USERS
- VLAN 30 — DMZ
- VLAN 40 — SECOPS
- VLAN 50 — SERVERS
- VLAN 60 — REDTEAM

OPNsense is the Layer-3 enforcement point. `SW-Lab-01` provides Layer-2 separation and VLAN transport.

## Dedicated Management Plane

`MGMT-01` (10.10.10.5), `MGMT-BACKUP` (10.10.10.6 when locally attached), OPNsense management (10.10.10.1), `SW-Lab-01` management (10.10.10.2), `ENTHOST-01` (10.10.10.3), and `SECHOST-01` (10.10.10.4) use VLAN 10 MANAGEMENT.

`MGMT-01` is the primary dedicated management system. `MGMT-BACKUP` provides a secondary administrative path when required. The baseline firewall policy permits MANAGEMENT to administer OPNsense over HTTPS and reach lab networks while blocking other unnecessary access to the firewall itself.

## SW-Lab-01 Management Hardening — Completed August 20, 2026

The Aruba J9774A management/security baseline has been implemented and validated:

- Management address: 10.10.10.2/24 on VLAN 10 MANAGEMENT
- Default gateway: 10.10.10.1
- Current firmware: `YA.16.10.0010`
- Firmware upgrade completed through a validated TFTP workflow
- Running configuration saved and an off-switch configuration backup exported
- Time synchronization configured and verified
- SNMPv3 enabled with a dedicated authenticated monitoring user
- SNMPv3 monitoring query succeeded from the management environment
- Legacy SNMPv1/v2c access using the former public community failed after hardening, while SNMPv3 continued to succeed
- Final saved configuration survived reboot, including management addressing, VLAN state, SNMPv3 hardening, and time synchronization

Authentication/privacy passwords, community strings, serial numbers, and unnecessary MAC addresses are excluded from public evidence.

## Current Switch-Port Security/Placement Baseline

- Port 1 — OPNsense trunk; tagged VLANs 10/20/30/40/50/60
- Port 6 — `MGMT-BACKUP`; untagged VLAN 10 MANAGEMENT
- Port 7 — `ENTHOST-01` physical attachment; final VLAN-aware Proxmox trunk/tagging configuration remains pending validation
- Port 8 — `MGMT-01`; untagged VLAN 10 MANAGEMENT
- Ports 2–5 and 9+ — no permanent role currently assigned

## Restricted User / Service / Attacker Zones

USERS, DMZ, SERVERS, and REDTEAM share the same core control pattern:

1. Permit DNS to OPNsense on TCP/UDP 53.
2. Block other access to OPNsense itself.
3. Block initiation toward `LAB_NETWORKS`.
4. Permit outbound/Internet traffic.

This creates an outbound-capable but laterally restricted baseline for those zones.

## SECOPS Administrative Reach

SECOPS is a trusted security-administration zone. Its baseline rules permit DNS to OPNsense, HTTPS administration of OPNsense, reachability to `LAB_NETWORKS`, and outbound/Internet access. Other access to the firewall itself is blocked by an explicit rule.

## Firewall Alias for Internal Networks

`LAB_NETWORKS` contains all six internal lab networks:

- 10.10.10.0/24
- 10.10.20.0/24
- 10.10.30.0/24
- 10.10.40.0/24
- 10.10.50.0/24
- 10.10.60.0/24

## DNS and DHCP Separation

Dnsmasq provides DHCP only on USERS, SECOPS, and REDTEAM. Its DNS function is disabled with Listen Port 0. Unbound remains the primary recursive/caching DNS resolver on port 53.

Dynamic pools are 10.10.20.100–199, 10.10.40.100–199, and 10.10.60.100–199. The authoritative host/IP baseline separately defines `WIN11-01` at 10.10.20.10 and `REDTEAM-01` at 10.10.60.10 outside those pools.

## WAN Exposure Control

OPNsense is directly connected to the ISP. The documented baseline does not expose the OPNsense WebUI or SSH on WAN. Unsolicited WAN ingress remains blocked unless a future lab explicitly introduces a tightly scoped published service/NAT rule.

WAN-addressing details are ISP-supplied and are not hard-coded in the public repository. WAN private-network and bogon options should follow the actual ISP handoff rather than an obsolete assumption that OPNsense sits behind a consumer router.

## Red-Team Isolation

`REDTEAM-01` is the bare-metal Kali Redteam Host at 10.10.60.10 on VLAN 60 REDTEAM and should be treated as attacker-controlled during exercises. Its baseline policy allows DNS and outbound Internet but blocks initiation to other lab VLANs and blocks management access to OPNsense.

## Host Telemetry and SIEM/XDR

Wazuh is the baseline centralized SIEM/XDR platform. The updated topology defines host telemetry from `WIN11-01`, `DC-01`, `LINUX-01`, `FILE-01`, and `WEB-01` to Wazuh on VLAN 40.

`DC-02` is a planned/rotational second domain controller and DNS server. It joins the telemetry design only while deployed and after its Wazuh agent is installed and verified.

## Passive Network Security Monitoring

Suricata and Zeek share the Network Security Monitoring VM. Selected switch traffic can be mirrored to a separate passive sensor NIC:

- Suricata — signature-based IDS inspection
- Zeek — network/protocol metadata and investigation context

The mirror/SPAN destination port is not finalized and must not be invented in the repository.

## DFIR / Threat Hunting

Velociraptor is kept separate from the SIEM and network sensor workload. It provides endpoint visibility, artifact collection, live response, triage, threat hunting, and digital-forensics workflows.

## Configuration / Evidence Hygiene

The public repository should not contain passwords or password hashes, API keys or tokens, certificates/private keys, pre-shared keys, VPN peer secrets, recovery codes, SNMP community strings, SNMPv3 authentication/privacy secrets, complete MAC addresses unless intentionally required, or public/ISP-assigned WAN information.

Lab VLAN IDs/subnets, OPNsense `.1` gateways, authoritative private host/IP assignments, `SW-Lab-01` management address, firmware version, and final switch-port roles are architectural evidence and can remain visible.

## Controls Requiring Future Live Evidence

The following remain pending and should be added only after verification:

- Proxmox firewall rules
- host-based firewall policy
- centralized backups / backup retention
- final Aruba mirror destination port
- syslog destinations
- unused-port shutdown policy
- final VM-by-VM endpoint hardening
- final `ENTHOST-01` VLAN-aware bridge/trunk configuration on port 7
- `DC-02` vCPU, storage, deployment, and resulting live security/telemetry controls

## Source Basis

- *OPNsense Home Lab Configuration — UPDATED* — firewall policy, DHCP/DNS split, aliases, direct ISP WAN exposure, and switch integration.
- *Authoritative VLAN Designations — final* — host/IP assignments.
- *Home Lab Network Topology Overview — UPDATED* — segmentation, telemetry, and security-zone intent.
- *Home Lab Hardware Inventory — UPDATED* — security tooling roles, management separation, and baseline memory plans.
- *Aruba 2530 Management and Configuration Guide for AOS-S 16.10* — SNMPv3, time synchronization, firmware, configuration persistence, and management capabilities.
- Verified SW-Lab-01 live configuration and validation evidence captured August 20, 2026.
