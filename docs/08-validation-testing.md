# Validation Testing

## Validation Principle

Documentation should prove that the intended architecture and firewall policy are actually enforced. Each validation item should record the source VLAN/system, destination, expected behavior, actual behavior, and supporting endpoint plus network/security evidence.

The current switch-port baseline now includes ports 1, 6, 7, and 8. Port 1 is the OPNsense trunk, port 6 is `MGMT-BACKUP`, port 7 is `ENTHOST-01`, and port 8 is `MGMT-01`. Final Proxmox VLAN-aware trunk validation on port 7 remains pending.

## Current Acceptance Tests

| Test | Expected result | Current evidence status |
|---|---|---|
| `MGMT-01` (10.10.10.5) on port 8 → 10.10.10.1 | Ping/WebUI succeeds | VLAN 10 management path established; dedicated connectivity evidence is not part of the final public Aruba evidence set |
| `MGMT-01` → 10.10.10.2 | Aruba WebUI/ping succeeds | Management connectivity verified; dedicated connectivity evidence is not part of the final public Aruba evidence set |
| `MGMT-BACKUP` (10.10.10.6 when local) on port 6 → `MGMT-01` / switch management | VLAN 10 connectivity succeeds | PASS; port 6 untagged VLAN 10 and local reachability previously validated |
| SW-Lab-01 firmware | Switch boots and operates on `YA.16.10.0010` | PASS; post-upgrade reboot validation completed; sanitized before/after firmware evidence uploaded |
| SW-Lab-01 SNMPv3 | Authenticated SNMPv3 query succeeds | PASS; validated from management environment; sanitized success evidence uploaded |
| SW-Lab-01 legacy SNMPv2c/public | Legacy query fails after SNMPv3 hardening | PASS; negative validation completed; repository evidence shows the legacy public community removed |
| SW-Lab-01 persistence | Management/VLAN/SNMPv3/time-sync state survives saved configuration and reboot | PASS; verified; dedicated persistence screenshot is not part of the final public evidence set |
| OPNsense Interfaces → Overview | Six VLAN interfaces enabled with `.1/24` addresses | Capture after final activation/verification of VLANs 20–60 |
| USERS DHCP | Lease in 10.10.20.100–199; gateway/DNS through 10.10.20.1 | Pending endpoint/access-port assignment |
| SECOPS DHCP | Lease in 10.10.40.100–199; gateway/DNS through 10.10.40.1 | Pending endpoint/access-port assignment |
| REDTEAM DHCP | Lease in 10.10.60.100–199; gateway/DNS through 10.10.60.1 | Pending final `REDTEAM-01` VLAN 60 attachment/validation |
| USERS/DMZ/SERVERS/REDTEAM → other lab VLANs | Blocked when initiated from restricted zone | Validate when endpoints exist |
| MANAGEMENT/SECOPS → lab networks | Allowed | Validate when endpoints exist |
| Restricted zones → OPNsense WebUI | Blocked; DNS to firewall remains allowed | Validate with endpoint test + Firewall Live View |
| Outbound Internet | Allowed per zone policy | Validate after DHCP/static clients are attached |

## Authoritative Static Host Validation Targets

These addresses should be validated when their systems are deployed and attached to the correct VLAN. A successful ping alone does not prove full system configuration; record VLAN placement, address/gateway settings, and expected firewall behavior as part of each acceptance test.

| Host | Address | VLAN | Validation status |
|---|---|---:|---|
| `opnsense-fw` / `OPNsense-FW` | 10.10.10.1 | 10 | Management gateway; final OPNsense evidence still needed |
| `SW-Lab-01` / `SW-LAB-01` | 10.10.10.2 | 10 | Management, firmware, SNMPv3, time-sync, and persistence validation completed; final sanitized evidence set uploaded |
| `ENTHOST-01` | 10.10.10.3 | 10 | Physical attachment on port 7 established; Proxmox management/VLAN-aware bridge validation still pending |
| `SECHOST-01` | 10.10.10.4 | 10 | Validate after Proxmox management placement is complete |
| `MGMT-01` | 10.10.10.5 | 10 | Port 8 untagged VLAN 10; management path established |
| `MGMT-BACKUP` | 10.10.10.6 when locally attached | 10 | PASS for local VLAN 10 reachability on port 6 |
| `WIN11-01` | 10.10.20.10 | 20 | Validate after VM/VLAN attachment |
| `WEB-01` | 10.10.30.10 | 30 | Validate after VM/VLAN attachment |
| `DC-01` | 10.10.50.10 | 50 | Validate after VM/VLAN attachment |
| `DC-02` | 10.10.50.20 | 50 | Planned/rotational second DC + DNS; validate after deployment |
| `FILE-01` | 10.10.50.30 | 50 | Validate after VM/VLAN attachment |
| `LINUX-01` | 10.10.50.40 | 50 | Validate after VMware-to-Proxmox migration and VLAN placement |
| `REDTEAM-01` | 10.10.60.10 | 60 | Validate after final VLAN 60 attachment |

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

## Aruba Validation Evidence - Uploaded August 20, 2026

The final sanitized Aruba evidence set is stored under `screenshots/aruba/`:

- [`01-switch-baseline-01-trunk-vlans.png`](../screenshots/aruba/01-switch-baseline-01-trunk-vlans.png) - port 1 participates in VLANs 10/20/30/40/50/60
- [`02-switch-baseline-02-vlan10-membership.png`](../screenshots/aruba/02-switch-baseline-02-vlan10-membership.png) - VLAN 10 shows port 1 tagged and ports 6/7/8 untagged
- [`03-switch-baseline-03-management-ip-gateway.png`](../screenshots/aruba/03-switch-baseline-03-management-ip-gateway.png) - switch management 10.10.10.2/24 with gateway 10.10.10.1
- [`04-switch-baseline-04-firmware-before.png`](../screenshots/aruba/04-switch-baseline-04-firmware-before.png) - pre-upgrade software revision YA.16.06.0006
- [`05-switch-baseline-05-firmware-after.png`](../screenshots/aruba/05-switch-baseline-05-firmware-after.png) - post-upgrade software revision YA.16.10.0010
- [`06-switch-baseline-06-snmpv3-success.png`](../screenshots/aruba/06-switch-baseline-06-snmpv3-success.png) - successful SNMPv3 monitoring query from the management environment
- [`07-switch-baseline-07-public-community-removed.png`](../screenshots/aruba/07-switch-baseline-07-public-community-removed.png) - legacy public SNMP community no longer present
- [`08-switch-baseline-08-sntp-status.png`](../screenshots/aruba/08-switch-baseline-08-sntp-status.png) - SNTP configured in unicast mode with current switch time displayed

The final public set intentionally omits lower-value/intermediate screenshots. The firmware evidence was sanitized before publication to obscure personal contact/location information, the switch base MAC address, and serial number. Authentication/privacy passwords and SNMP community strings are not published.
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
