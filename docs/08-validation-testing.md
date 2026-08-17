# Validation Testing

## Validation Principle

Documentation should prove that the intended architecture and firewall policy are actually enforced. Each validation item should record the source VLAN/system, destination, expected behavior, actual behavior, and supporting endpoint plus network/security evidence.

Because switch/access-port assignments other than port 8 remain TBD, MANAGEMENT can be tested immediately. The remaining VLANs should be revalidated when their Proxmox or access-port attachment becomes permanent.

## Current Acceptance Tests

| Test | Expected result | Current evidence status |
|---|---|---|
| MANAGEMENT PC on port 8 → 10.10.10.1 | Ping/WebUI succeeds | VLAN 10 reachability was previously recovered; recapture clean final evidence on port 8 |
| MANAGEMENT PC → 10.10.10.2 | Aruba WebUI/ping succeeds | Expected from VLAN 10 management design; capture clean evidence |
| OPNsense Interfaces → Overview | Six VLAN interfaces enabled with `.1/24` addresses | Capture after final activation/verification of VLANs 20–60 |
| USERS DHCP | Lease in 10.10.20.100–199; gateway/DNS through 10.10.20.1 | Pending endpoint/access-port assignment |
| SECOPS DHCP | Lease in 10.10.40.100–199; gateway/DNS through 10.10.40.1 | Pending endpoint/access-port assignment |
| REDTEAM DHCP | Lease in 10.10.60.100–199; gateway/DNS through 10.10.60.1 | Pending endpoint/access-port assignment |
| USERS/DMZ/SERVERS/REDTEAM → other lab VLANs | Blocked when initiated from restricted zone | Validate when endpoints exist |
| MANAGEMENT/SECOPS → lab networks | Allowed | Validate when endpoints exist |
| Restricted zones → OPNsense WebUI | Blocked; DNS to firewall remains allowed | Validate with endpoint test + Firewall Live View |
| Outbound Internet | Allowed per zone policy | Validate after DHCP/static clients are attached |

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

The current source specifically calls for:

`validation-management-connectivity.png`

The capture should show reachability from the Management Host to:

- 10.10.10.1 — OPNsense MANAGEMENT gateway/WebUI
- 10.10.10.2 — `SW-Lab-01` management interface

Do not show credentials.

## Aruba Validation Evidence

Two switch captures are directly tied to the OPNsense trunk/access design:

- `aruba-port1-opnsense-trunk.png` — port 1 Tagged in VLANs 10/20/30/40/50/60
- `aruba-port8-management-access.png` — VLAN 10 shows port 1 Tagged and port 8 Untagged

## OPNsense Rule Validation

At minimum, capture the complete ordered rule sets for MANAGEMENT and SECOPS and one representative restricted zone. Validate denied flows with **Firewall → Log Files → Live View** so the repository contains both endpoint behavior and firewall evidence.

## Troubleshooting Sequence

If validation fails, use the source-preserved sequence before changing architecture:

- If a VLAN is saved but missing from Interfaces → Overview, verify the runtime interface/device first.
- If MANAGEMENT exists but ping fails, inspect firewall rule order.
- If ping succeeds but packet capture is empty, confirm capture is running on the interface that actually sees the packets and apply an ICMP filter when appropriate.
- Use Firewall → Log Files → Live View to identify the rule matching a denied flow.
- Apply pending firewall/interface changes before retesting; unapplied changes can produce misleading results.

## Do Not Mark Pending Tests as Successful

VLANs 20/30/40/50/60 still require permanent endpoint/host attachment and clean validation evidence. The repository should clearly label those tests **Pending** until the relevant systems are connected and the expected allow/block behavior is demonstrated.

## Source Basis

- *OPNsense Home Lab Configuration, Phases 3–12* — current acceptance-test matrix, troubleshooting sequence, and evidence filenames.
- *Home Lab Network Topology Overview* — intended traffic and security-zone relationships.
