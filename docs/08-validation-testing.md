# Validation Testing

## Validation Principle

Documentation should prove that the intended architecture and firewall policy are actually enforced. Each validation item should record the source VLAN/system, destination, expected behavior, actual behavior, and supporting endpoint plus network/security evidence.

The current switch-port baseline now includes ports 1, 6, 7, and 8. Port 1 is the OPNsense trunk, port 6 is `MGMT-BACKUP`, port 7 is `ENTHOST-01`, and port 8 is `MGMT-01`. Final Proxmox VLAN-aware trunk validation on port 7 remains pending.

## Current Acceptance Tests

| Test | Expected result | Current evidence status |
|---|---|---|
| `MGMT-01` (10.10.10.6) on port 8 → 10.10.10.1 | Ping/WebUI succeeds | VLAN 10 management path established; final clean evidence captured/pending upload |
| `MGMT-01` → 10.10.10.2 | Aruba WebUI/ping succeeds | Management connectivity verified; evidence captured/pending upload |
| `MGMT-BACKUP` (10.10.10.10 when local) on port 6 → `MGMT-01` / switch management | VLAN 10 connectivity succeeds | PASS; port 6 untagged VLAN 10 and local reachability previously validated |
| SW-Lab-01 firmware | Switch boots and operates on `YA.16.10.0010` | PASS; post-upgrade reboot validation completed; screenshot pending upload |
| SW-Lab-01 SNMPv3 | Authenticated SNMPv3 query succeeds | PASS; validated from management environment; screenshot pending upload |
| SW-Lab-01 legacy SNMPv2c/public | Legacy query fails after SNMPv3 hardening | PASS; negative validation completed; screenshot pending upload |
| SW-Lab-01 persistence | Management/VLAN/SNMPv3/time-sync state survives saved configuration and reboot | PASS; verified; screenshot evidence pending upload |
| OPNsense Interfaces → Overview | Six VLAN interfaces enabled with `.1/24` addresses | Capture after final activation/verification of VLANs 20–60 |
| USERS DHCP | Lease in 10.10.20.100–199; gateway/DNS through 10.10.20.1 | Pending endpoint/access-port assignment |
| SECOPS DHCP | Lease in 10.10.40.100–199; gateway/DNS through 10.10.40.1 | Pending endpoint/access-port assignment |
| REDTEAM DHCP | Lease in 10.10.60.100–199; gateway/DNS through 10.10.60.1 | Pending final `READTEAM-01` VLAN 60 attachment/validation |
| USERS/DMZ/SERVERS/REDTEAM → other lab VLANs | Blocked when initiated from restricted zone | Validate when endpoints exist |
| MANAGEMENT/SECOPS → lab networks | Allowed | Validate when endpoints exist |
| Restricted zones → OPNsense WebUI | Blocked; DNS to firewall remains allowed | Validate with endpoint test + Firewall Live View |
| Outbound Internet | Allowed per zone policy | Validate after DHCP/static clients are attached |

## Authoritative Static Host Validation Targets

These addresses should be validated when their systems are deployed and attached to the correct VLAN. A successful ping alone does not prove full system configuration; record VLAN placement, address/gateway settings, and expected firewall behavior as part of each acceptance test.

| Host | Address | VLAN | Validation status |
|---|---|---:|---|
| `opnsense-fw` / `OPNsense-FW` | 10.10.10.1 | 10 | Management gateway; final OPNsense evidence still needed |
| `SW-Lab-01` / `SW-LAB-01` | 10.10.10.2 | 10 | Management, firmware, SNMPv3, time-sync, and persistence validation completed; screenshots pending upload |
| `ENTHOST-01` | 10.10.10.3 | 10 | Physical attachment on port 7 established; Proxmox management/VLAN-aware bridge validation still pending |
| `SECHOST-01` | 10.10.10.4 | 10 | Validate after Proxmox management placement is complete |
| `MGMT-01` | 10.10.10.6 | 10 | Port 8 untagged VLAN 10; management path established |
| `MGMT-BACKUP` | 10.10.10.10 when locally attached | 10 | PASS for local VLAN 10 reachability on port 6 |
| `WIN11-01` | 10.10.20.10 | 20 | Validate after VM/VLAN attachment |
| `WEB-01` | 10.10.30.10 | 30 | Validate after VM/VLAN attachment |
| `DC-01` | 10.10.50.10 | 50 | Validate after VM/VLAN attachment |
| `DC-02` | 10.10.50.20 | 50 | Planned/rotational second DC + DNS; validate after deployment |
| `FILE-01` | 10.10.50.30 | 50 | Validate after VM/VLAN attachment |
| `LINUX-01` | 10.10.50.40 | 50 | Validate after VMware-to-Proxmox migration and VLAN placement |
| `READTEAM-01` | 10.10.60.10 | 60 | Validate after final VLAN 60 attachment |

## Firewall Validation Matrix

| Source zone | DNS to OPNsense | HTTPS WebUI to OPNsense | Other access to OPNsense | Other lab VLANs | Internet/outbound |
|---|---|---|---|---|---|
| MANAGEMENT | Allow | Allow | Block | Allow | Allow |
| USERS | Allow | Block | Block | Block | Allow |
| DMZ | Allow | Block | Block | Block | Allow |
| SECOPS | Allow | Allow | Block | Allow | Allow |
| SERVERS | Allow | Block | Block | Block | Allow |
| REDTEAM | Allow | Block | Block | Block | Allow |

## Evidence Pattern

For each test, record:

1. Source system and VLAN
2. Destination system/IP and VLAN
3. Expected result
4. Test method
5. Actual result
6. Endpoint screenshot/output
7. OPNsense Firewall Live View or other network evidence showing the matching pass/block
8. Final result: PASS / FAIL / NEEDS INVESTIGATION

## Required Management Validation Evidence

`validation-management-connectivity.png` should show reachability from `MGMT-01` to:

- 10.10.10.1 — OPNsense MANAGEMENT gateway/WebUI
- 10.10.10.2 — `SW-Lab-01` management interface

Do not show credentials.

## Aruba Validation Evidence — Captured August 20, 2026

The following categories were captured and are pending sanitized upload under `screenshots/aruba/`:

- switch identity / firmware state
- management IP 10.10.10.2/24 and default gateway 10.10.10.1
- port 1 tagged in VLANs 10/20/30/40/50/60
- port 6 `MGMT-BACKUP` VLAN 10 access state as applicable
- port 7 `ENTHOST-01` physical assignment
- port 8 `MGMT-01` VLAN 10 access state
- firmware transfer / upgraded firmware validation
- pre-hardening SNMP state
- successful SNMPv3 validation
- failed legacy SNMPv2c/public validation after hardening
- saved configuration / reboot persistence
- final time synchronization evidence (`13-time-sync.png`)

Screenshots must not expose passwords, SNMP authentication/privacy secrets, community strings, serial numbers, unnecessary complete MAC addresses, or public WAN information.

## OPNsense Rule Validation

At minimum, capture the complete ordered rule sets for MANAGEMENT and SECOPS and one representative restricted zone. Validate denied flows with **Firewall → Log Files → Live View** so the repository contains both endpoint behavior and firewall evidence.

## WAN Validation

OPNsense is directly ISP-facing. Validate WAN status from OPNsense without publishing public/ISP-assigned WAN details. Do not rely on any test or screenshot that depicts a consumer router as part of the current lab path.

## Troubleshooting Sequence

If validation fails, use the source-preserved sequence before changing architecture:

- If a VLAN is saved but missing from Interfaces → Overview, verify the runtime interface/device first.
- If MANAGEMENT exists but ping fails, inspect firewall rule order.
- If ping succeeds but packet capture is empty, confirm capture is running on the interface that actually sees the packets and apply an ICMP filter when appropriate.
- Use Firewall → Log Files → Live View to identify the rule matching a denied flow.
- Apply pending firewall/interface changes before retesting; unapplied changes can produce misleading results.

## Do Not Mark Pending Tests as Successful

The Aruba management/firmware/SNMPv3 baseline is complete, but guest VLAN acceptance testing remains separate. VLANs 20/30/40/50/60 still require their intended VM/host placement and end-to-end firewall validation. `ENTHOST-01` is physically attached on port 7, but its final Proxmox VLAN-aware bridge/trunk design remains pending. `DC-02` remains planned/rotational until deployed and validated.

## Source Basis

- *OPNsense Home Lab Configuration — UPDATED* — current acceptance-test matrix, troubleshooting sequence, evidence filenames, and direct ISP WAN.
- *Authoritative VLAN Designations — final* — host/IP validation targets.
- *Home Lab Network Topology Overview — UPDATED* — intended traffic and security-zone relationships.
- *MGMT-01 Remote RDP Resolution Summary* — verified port 6/8 VLAN 10 connectivity.
- Verified SW-Lab-01 live configuration and validation evidence captured August 20, 2026.
