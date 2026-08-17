# Virtual Machines

## Documentation Status

This page records the baseline virtual workloads defined by the authoritative hardware inventory. Installation state, operating-system versions, hostnames beyond those explicitly documented, storage sizes, CPU allocations, and live IP addresses should be added only after each VM is deployed and verified.

## Enterprise Workloads

| VM | RAM | Network | Primary function | Purpose |
|---|---:|---|---|---|
| DC01 — Windows Server | 4 GB | VLAN 50 — SERVERS | Active Directory Domain Services + DNS | Provides the lab domain, centralized authentication, directory services, Group Policy testing, domain-joined identity, and AD-integrated DNS. |
| Ubuntu Server | 4 GB | VLAN 50 — SERVERS | Linux server | Provides Linux administration, SSH, services, hardening, logging, automation, and security exercises. |
| Web Server | 2 GB | VLAN 30 — DMZ | DMZ web application/server | Hosts a web service isolated in the DMZ for administration, hardening, logging, monitoring, and controlled attack/defense exercises. |
| Windows 11 Workstation | 8 GB | VLAN 20 — USERS | Domain-joined user endpoint | Simulates a normal enterprise user workstation for Active Directory, Group Policy, endpoint logging, Sysmon, Wazuh agent deployment, and red-team/blue-team exercises. |
| File Server | 4 GB | VLAN 50 — SERVERS | Centralized file services | Provides SMB/file shares, NTFS permissions, user/group access testing, audit logging, file-security scenarios, and centralized storage exercises. |

## Security Workloads

| VM | RAM | Network | Core tools / role | Purpose |
|---|---:|---|---|---|
| Wazuh SIEM/XDR VM | 8 GB | VLAN 40 — SECOPS | Wazuh manager, indexer, dashboard | Centralizes endpoint/server telemetry, correlates alerts, performs file-integrity monitoring, security configuration assessment, vulnerability visibility, and investigation. |
| Network Security Monitoring VM | 8 GB | VLAN 40 — SECOPS + passive sensor path | Suricata + Zeek | Receives mirrored traffic, performs signature-based IDS inspection, and creates detailed network/protocol metadata for detection and investigation. |
| DFIR / Threat Hunting VM | 4 GB | VLAN 40 — SECOPS | Velociraptor / investigation tools | Supports endpoint visibility, artifact collection, live response, triage, threat hunting, and digital-forensics workflows. |

## Non-VM Systems Relevant to Workload Testing

| System | Network | Role |
|---|---|---|
| Management Host | VLAN 10 — MANAGEMENT | Trusted administrative workstation for infrastructure management |
| Redteam Host | VLAN 60 — REDTEAM | Bare-metal Kali Linux attacker simulation / offensive-security platform |

## Telemetry Relationships

The topology source defines the following intended relationships:

- Windows 11 → Wazuh agent/log telemetry
- DC01 → Wazuh agent/log telemetry
- Ubuntu Server → Wazuh agent/Linux telemetry
- File Server → Wazuh agent/log telemetry
- Web Server → Wazuh agent/Linux or server telemetry
- Selected network traffic → `SW-Lab-01` mirror/SPAN → Suricata + Zeek passive sensor
- Suspicious endpoint → Velociraptor / DFIR triage and collection

## Addressing Expectations

Network gateways are provided by OPNsense:

| VLAN | Gateway | Addressing model |
|---:|---|---|
| 20 USERS | 10.10.20.1 | DHCP pool 10.10.20.100–199 |
| 30 DMZ | 10.10.30.1 | Static service addressing |
| 40 SECOPS | 10.10.40.1 | DHCP pool 10.10.40.100–199 for applicable security endpoints/workloads |
| 50 SERVERS | 10.10.50.1 | Static server addressing |

The source does not assign final individual guest IP addresses, so none are documented here.

## Deployment Tracking

As each VM is installed, this page can be extended with a short **Deployment Status** entry containing only verified facts such as system name, role, VLAN, OS/version, allocated RAM, and evidence links. Do not publish passwords, domain secrets, activation keys, private keys, or other authentication material.

## Source Basis

- *Home Lab Hardware Inventory* — VM list, RAM allocations, tools, and purposes.
- *Home Lab Network Topology Overview* — VLAN placement and telemetry flow.
- *OPNsense Home Lab Configuration, Phases 3–12* — gateway and DHCP design.
