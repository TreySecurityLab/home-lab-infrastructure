# Virtual Machines

## Documentation Status

This page records the baseline virtual workloads defined by the updated hardware inventory and the authoritative hostname/IP assignments. Installation state, operating-system versions, storage sizes, CPU allocations, and live configuration should be added only after each VM is deployed and verified.

## Enterprise Workloads

| Hostname | Address | RAM | Network | Primary function | Status / purpose |
|---|---|---:|---|---|---|
| `DC-01` | 10.10.50.10 | 4 GB | VLAN 50 — SERVERS | Active Directory Domain Services + DNS | Baseline planned enterprise workload |
| `DC-02` | 10.10.50.20 | TBD | VLAN 50 — SERVERS | Exact services TBD | Intentionally planned; exact resource allocation and service scope are not yet defined in the five-source baseline |
| `FILE-01` | 10.10.50.30 | 4 GB | VLAN 50 — SERVERS | Centralized file services | SMB/file shares, NTFS permissions, access testing, audit logging, and storage exercises |
| `LINUX-01` | 10.10.50.40 | 4 GB | VLAN 50 — SERVERS | Ubuntu/Linux server | Linux administration, SSH, services, hardening, logging, automation, and security exercises |
| `WEB-01` | 10.10.30.10 | 2 GB | VLAN 30 — DMZ | DMZ web application/server | Isolated web service for hardening, logging, monitoring, and controlled attack/defense exercises |
| `WIN11-01` | 10.10.20.10 | 8 GB | VLAN 20 — USERS | Domain-joined user endpoint | Active Directory, Group Policy, endpoint logging, Sysmon, Wazuh, and red/blue-team exercises |

## Security Workloads

| VM | RAM | Network | Core tools / role | Purpose |
|---|---:|---|---|---|
| Wazuh SIEM/XDR VM | 8 GB | VLAN 40 — SECOPS | Wazuh manager, indexer, dashboard | Centralized endpoint/server telemetry, alert correlation, FIM, configuration assessment, vulnerability visibility, and investigation |
| Network Security Monitoring VM | 8 GB | VLAN 40 — SECOPS + passive sensor path | Suricata + Zeek | Signature-based IDS inspection and network/protocol metadata from mirrored traffic |
| DFIR / Threat Hunting VM | 4 GB | VLAN 40 — SECOPS | Velociraptor / investigation tools | Endpoint visibility, artifact collection, live response, triage, threat hunting, and digital-forensics workflows |

The final authoritative hostname/IP source does not currently assign static addresses to VLAN 40 security workloads. Their addresses remain pending rather than inferred.

## Non-VM Systems Relevant to Workload Testing

| System | Address | Network | Role |
|---|---|---|---|
| `MGMT-01` | 10.10.10.6 | VLAN 10 — MANAGEMENT | Trusted administrative workstation for infrastructure management |
| `ENTHOST-01` | 10.10.10.3 | VLAN 10 — MANAGEMENT | Enterprise Proxmox host |
| `SECHOST-01` | 10.10.10.4 | VLAN 10 — MANAGEMENT | Security Proxmox host |
| `KALI-01` | 10.10.60.10 | VLAN 60 — REDTEAM | Bare-metal Kali Linux attacker-simulation / offensive-security platform |

## Telemetry Relationships

The updated topology source defines the following intended relationships:

- `WIN11-01` → Wazuh agent/log telemetry
- `DC-01` → Wazuh agent/log telemetry
- `LINUX-01` → Wazuh agent/Linux telemetry
- `FILE-01` → Wazuh agent/log telemetry
- `WEB-01` → Wazuh agent/Linux or server telemetry
- Selected network traffic → `SW-Lab-01` mirror/SPAN → Suricata + Zeek passive sensor
- Suspicious endpoint → Velociraptor / DFIR triage and collection

`DC-02` is not added to the telemetry list until its exact service role and deployment are defined and verified.

## Network Gateways and Addressing

| VLAN | Gateway | Authoritative named workload addresses | DHCP model |
|---:|---|---|---|
| 20 USERS | 10.10.20.1 | `WIN11-01` 10.10.20.10 | DHCP pool 10.10.20.100–199 also exists for dynamic endpoints |
| 30 DMZ | 10.10.30.1 | `WEB-01` 10.10.30.10 | Static service addressing |
| 40 SECOPS | 10.10.40.1 | None defined in final hostname/IP source | DHCP pool 10.10.40.100–199 |
| 50 SERVERS | 10.10.50.1 | `DC-01` .10, `DC-02` .20, `FILE-01` .30, `LINUX-01` .40 | Static server addressing |
| 60 REDTEAM | 10.10.60.1 | `KALI-01` 10.10.60.10 | DHCP pool 10.10.60.100–199 also exists for dynamic endpoints |

## Deployment Tracking

As each VM is installed, extend this page only with verified facts such as system name, role, VLAN, OS/version, allocated RAM/vCPU, storage, deployment state, and evidence links. Do not publish passwords, domain secrets, activation keys, private keys, or other authentication material.

## `DC-02` Source Gap

The authoritative VLAN source assigns `DC-02` to 10.10.50.20, and its planned inclusion has been explicitly confirmed. The updated hardware inventory and topology do not yet define its RAM/vCPU allocation or exact role. Those source documents should be revised before `DC-02` becomes an operational prerequisite or is counted in the enterprise-host memory plan.

## Source Basis

- *Home Lab Hardware Inventory — UPDATED* — baseline VM list, RAM allocations, tools, and purposes.
- *Authoritative VLAN Designations — final* — authoritative hostnames/inventory labels and IP assignments, including `DC-02`.
- *Home Lab Network Topology Overview — UPDATED* — VLAN placement and telemetry flow.
- *OPNsense Home Lab Configuration — UPDATED* — gateway and DHCP design.
