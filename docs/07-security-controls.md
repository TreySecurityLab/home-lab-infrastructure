# Security Controls

## Control Model

The lab is built around segmentation, least exposure, dedicated administration, centralized telemetry, and explicit validation. This page documents controls supported by the current source material and separates them from controls that remain planned or require live evidence.

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

`MGMT-01` (10.10.10.6), OPNsense management (10.10.10.1), `SW-Lab-01` management (10.10.10.2), `ENTHOST-01` (10.10.10.3), and `SECHOST-01` (10.10.10.4) are placed on VLAN 10 MANAGEMENT.

The Management Host is not intended to be repurposed as a normal lab target. The baseline firewall policy permits MANAGEMENT to administer OPNsense over HTTPS and reach lab networks while blocking other unnecessary access to the firewall itself.

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

Dynamic pools are 10.10.20.100–199, 10.10.40.100–199, and 10.10.60.100–199. The authoritative host/IP source separately defines `WIN11-01` at 10.10.20.10 and `KALI-01` at 10.10.60.10 outside those pools.

## WAN Exposure Control

OPNsense is directly connected to the ISP. The documented baseline does not expose the OPNsense WebUI or SSH on WAN. Unsolicited WAN ingress remains blocked unless a future lab explicitly introduces a tightly scoped published service/NAT rule.

WAN-addressing details are ISP-supplied and are not hard-coded in the public repository. WAN private-network and bogon options should follow the actual ISP handoff rather than an obsolete assumption that OPNsense sits behind a consumer router.

## Red-Team Isolation

`KALI-01` is the bare-metal Kali Redteam Host at 10.10.60.10 on VLAN 60 REDTEAM and should be treated as attacker-controlled during exercises. Its baseline policy allows DNS and outbound Internet but blocks initiation to other lab VLANs and blocks management access to OPNsense.

## Host Telemetry and SIEM/XDR

Wazuh is the baseline centralized SIEM/XDR platform. The updated topology defines host telemetry from `WIN11-01`, `DC-01`, `LINUX-01`, `FILE-01`, and `WEB-01` to Wazuh on VLAN 40.

`DC-02` is intentionally planned but is not yet assigned a defined telemetry role because its exact services and deployment details remain pending in the current baseline sources.

## Passive Network Security Monitoring

Suricata and Zeek share the Network Security Monitoring VM. Selected switch traffic can be mirrored to a separate passive sensor NIC:

- Suricata — signature-based IDS inspection
- Zeek — network/protocol metadata and investigation context

The mirror/SPAN destination port is not finalized and must not be invented in the repository.

## DFIR / Threat Hunting

Velociraptor is kept separate from the SIEM and network sensor workload. It provides endpoint visibility, artifact collection, live response, triage, threat hunting, and digital-forensics workflows.

## Configuration / Evidence Hygiene

The public repository should not contain passwords or password hashes, API keys or tokens, certificates/private keys, pre-shared keys, VPN peer secrets, recovery codes, SNMP community strings, complete MAC addresses unless intentionally required, or public/ISP-assigned WAN information.

Lab VLAN IDs/subnets, OPNsense `.1` gateways, authoritative private host/IP assignments, `SW-Lab-01` management address, and final switch-port roles are architectural evidence and can remain visible.

## Controls Requiring Future Live Evidence

The sources do not yet establish the final live implementation of the following, so they should be added only after verification:

- Proxmox firewall rules
- host-based firewall policy
- centralized backups / backup retention
- final Aruba mirror destination port
- switch hardening beyond the documented VLAN/management baseline
- SNMPv3 deployment
- syslog destinations
- final NTP/SNTP configuration
- unused-port shutdown policy
- final VM-by-VM endpoint hardening
- `DC-02` exact services, RAM/vCPU, and resulting security/telemetry controls

## Source Basis

- *OPNsense Home Lab Configuration — UPDATED* — firewall policy, DHCP/DNS split, aliases, direct ISP WAN exposure, and current switch integration.
- *Authoritative VLAN Designations — final* — host/IP assignments.
- *Home Lab Network Topology Overview — UPDATED* — segmentation, telemetry, and security-zone intent.
- *Home Lab Hardware Inventory — UPDATED* — security tooling roles, management separation, and baseline memory plans.
