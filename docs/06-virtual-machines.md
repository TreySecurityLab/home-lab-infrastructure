# Virtual Machines

## Documentation Status

This page records the baseline virtual workloads defined by the updated hardware inventory and the authoritative hostname/IP assignments. Installation state, operating-system versions, storage sizes, CPU allocations, and live configuration should be added only after each VM is deployed and verified.

## Enterprise Workloads

| Hostname | Address | RAM | Network | Primary function | Status / purpose |
|---|---|---:|---|---|---|
| `DC-01` | 10.10.50.10 | 4 GB | VLAN 50 — SERVERS | Active Directory Domain Services + DNS | Baseline planned enterprise workload |
| `DC-02` | 10.10.50.20 | 3–4 GB | VLAN 50 — SERVERS | Second domain controller + DNS | Planned/rotational; deploy after `DC-01` is stable; not part of the normal always-on 22 GB allocation |
| `FILE-01` | 10.10.50.30 | 4 GB | VLAN 50 — SERVERS | Centralized file services | SMB/file shares, NTFS permissions, access testing, audit logging, and storage exercises |
| `LINUX-01` | 10.10.50.40 | 4 GB | VLAN 50 — SERVERS | Ubuntu/Linux server | Primary Linux/ChatGPT lab server after VMware-to-Proxmox migration and validation |
| `WEB-01` | 10.10.30.10 | 2 GB | VLAN 30 — DMZ | DMZ web application/server | Isolated web service for hardening, logging, monitoring, and controlled attack/defense exercises |
| `WIN11-01` | 10.10.20.10 | 8 GB | VLAN 20 — USERS | Domain-joined user endpoint | Active Directory, Group Policy, endpoint logging, Sysmon, Wazuh, and red/blue-team exercises |

The normal Enterprise Host always-on guest allocation remains 22 GB across `DC-01`, `LINUX-01`, `WEB-01`, `WIN11-01`, and `FILE-01`. `DC-02` is an additional rotational workload using approximately 3–4 GB only while active.

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
| `MGMT-01` | 10.10.10.5 | VLAN 10 — MANAGEMENT | Trusted primary administrative workstation for infrastructure management |
| `MGMT-BACKUP` | 10.10.10.6 when locally attached | VLAN 10 — MANAGEMENT | Backup/remote administrative laptop |
| `ENTHOST-01` | 10.10.10.3 | VLAN 10 — MANAGEMENT | Enterprise Proxmox host; physical switch attachment is SW-Lab-01 port 7 |
| `SECHOST-01` | 10.10.10.4 | VLAN 10 — MANAGEMENT | Security Proxmox host |
| `REDTEAM-01` | 10.10.60.10 | VLAN 60 — REDTEAM | Bare-metal Kali Linux attacker-simulation / offensive-security platform |

## Telemetry Relationships

The updated topology source defines the following intended relationships:

- `WIN11-01` → Wazuh agent/log telemetry after deployment and verification
- `DC-01` → Wazuh agent/log telemetry after deployment and verification
- `LINUX-01` → Wazuh agent/Linux telemetry after deployment and verification
- `FILE-01` → Wazuh agent/log telemetry after deployment and verification
- `WEB-01` → Wazuh agent/Linux or server telemetry after deployment and verification
- `DC-02` → Wazuh telemetry only while the rotational VM is deployed and after its agent is installed and verified
- Selected network traffic → `SW-Lab-01` mirror/SPAN → Suricata + Zeek passive sensor after the mirror path is configured and verified
- Suspicious endpoint → Velociraptor / DFIR triage and collection after the server and clients are operational

## Network Gateways and Addressing

| VLAN | Gateway | Authoritative named workload addresses | DHCP model |
|---:|---|---|---|
| 20 USERS | 10.10.20.1 | `WIN11-01` 10.10.20.10 | DHCP pool 10.10.20.100–199 also exists for dynamic endpoints |
| 30 DMZ | 10.10.30.1 | `WEB-01` 10.10.30.10 | Static service addressing |
| 40 SECOPS | 10.10.40.1 | None defined in final hostname/IP source | DHCP pool 10.10.40.100–199 |
| 50 SERVERS | 10.10.50.1 | `DC-01` .10, `DC-02` .20, `FILE-01` .30, `LINUX-01` .40 | Static server addressing |
| 60 REDTEAM | 10.10.60.1 | `REDTEAM-01` 10.10.60.10 | DHCP pool 10.10.60.100–199 also exists for dynamic endpoints |

## Deployment Tracking

As each VM is installed, extend this page only with verified facts such as system name, role, VLAN, OS/version, allocated RAM/vCPU, storage, deployment state, and evidence links. Do not publish passwords, domain secrets, activation keys, private keys, or other authentication material.

## `DC-02` Planned / Rotational Baseline

`DC-02` is intentionally part of the home-lab design at 10.10.50.20 on VLAN 50 SERVERS. It is a planned/rotational second domain controller and DNS server requiring approximately 3–4 GB RAM. It should be deployed only after `DC-01` is stable and should not be counted in the normal always-on 22 GB Enterprise VM allocation unless the memory plan is later revised and verified.

## Verified-State Rule

Planned workloads and telemetry relationships must not be presented as operational until the corresponding VM, network placement, service, agent, or sensor path has been installed, configured, tested, and verified.

## Source Basis

- `Home-Lab-Hardware-Inventory-final-2026-08-20.pdf` — baseline VM list, RAM allocations, tools, and purposes.
- `Authoritative-VLAN-Designations-final-2026-08-20.txt` — authoritative hostnames/inventory labels and IP assignments, including `DC-02`.
- `Home-Lab-Network-Topology-final-2026-08-20.pdf` — VLAN placement and telemetry flow.
- `OPNsense-Phases-3-12-Home-Lab-Configuration-final-2026-08-20.pdf` — gateway and DHCP design.
