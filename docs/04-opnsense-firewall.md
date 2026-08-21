# OPNsense Firewall

## Documentation Status

**Configured hostname:** `opnsense-fw`  
**Platform:** Bare-metal OPNsense / 8 GB RAM  
**Documented scope:** Phases 3ÃƒÂ¢Ã¢â€šÂ¬Ã¢â‚¬Å“12 ÃƒÂ¢Ã¢â€šÂ¬Ã¢â‚¬Â interfaces, VLANs, DHCP, DNS, aliases, firewall policy, and Aruba switch integration.

This page separates the current repository baseline from temporary recovery settings used while VLAN 10 was repaired. Recovery values are retained only as implementation history so they are not mistaken for permanent architecture.

## Role in the Architecture

OPNsense is directly connected to the ISP and is the Layer-3 gateway and policy-enforcement point for all six lab VLANs. It provides:

- ISP-facing WAN connectivity
- NAT
- 802.1Q VLAN termination
- Per-VLAN default gateways
- Inter-VLAN security policy
- DHCP for selected endpoint-oriented VLANs
- DNS service through Unbound
- Reusable firewall aliases
- Controlled outbound Internet access

`SW-Lab-01` remains Layer-2 only.

## Physical Interfaces

| Interface | Role | Current baseline |
|---|---|---|
| `re0` | WAN | Direct ISP-facing WAN; addressing supplied by the ISP and intentionally not hard-coded in public documentation |
| `ue0` | LAN/VLAN parent | Validated runtime parent toward `SW-Lab-01` port 1; carries VLAN tags 10/20/30/40/50/60 |

Earlier build notes referenced `ve0` or `vtnet1` during different hardware/interface-enumeration stages. The updated source identifies `ue0` as the validated runtime parent.

## WAN Baseline

- Internet / ISP service handoff connects directly to OPNsense WAN.
- WAN addressing is ISP-supplied; public or ISP-assigned values should not be hard-coded in GitHub.
- **Block private networks** should be configured according to the actual ISP handoff: enable it for a public WAN and disable it only when the ISP supplies private/CGNAT space.
- **Block bogon networks** should remain aligned with the actual WAN design and current OPNsense guidance.
- Do not expose the OPNsense WebUI or SSH on WAN. Unsolicited WAN ingress remains blocked unless a future lab deliberately introduces a tightly scoped published service/NAT rule.

## Basic System Configuration

| Setting | Documented value / action |
|---|---|
| Hostname | `opnsense-fw` |
| Domain | `lab.internal` |
| Timezone | Capture from the live system; the updated source does not specify a final value |
| WAN IPv4 | ISP-supplied / do not hard-code public details |
| Upstream router | ISP handoff; no separate consumer router is part of the lab topology |

## VLAN Devices

All six VLAN devices are created on the physical LAN/trunk parent.

| VLAN | Parent | Tag | Description |
|---:|---|---:|---|
| 10 | `ue0` | 10 | MANAGEMENT |
| 20 | `ue0` | 20 | USERS |
| 30 | `ue0` | 30 | DMZ |
| 40 | `ue0` | 40 | SECOPS |
| 50 | `ue0` | 50 | SERVERS |
| 60 | `ue0` | 60 | REDTEAM |

## Assigned VLAN Interfaces

Each VLAN device is assigned as an OPNsense interface and enabled with:

- IPv4 Configuration Type: **Static IPv4**
- IPv6 Configuration Type: **None** for the documented IPv4-only baseline
- IPv4 Upstream Gateway: **None** on internal VLAN interfaces
- Block private networks: **unchecked** on internal VLANs
- Block bogon networks: **unchecked** on internal VLANs

## VLAN Gateways

| Interface | Static IPv4 address | Role |
|---|---|---|
| MANAGEMENT | 10.10.10.1/24 | Administrative gateway |
| USERS | 10.10.20.1/24 | User-zone gateway |
| DMZ | 10.10.30.1/24 | DMZ gateway |
| SECOPS | 10.10.40.1/24 | Security-operations gateway |
| SERVERS | 10.10.50.1/24 | Server-zone gateway |
| REDTEAM | 10.10.60.1/24 | Red-team gateway |

The authoritative host/IP source uses 10.10.10.1 as the `OPNsense-FW` inventory address; the OPNsense configuration source identifies the configured hostname as `opnsense-fw`.

## DHCP and DNS Design

| VLAN | DHCP | Pool / policy |
|---:|---|---|
| 10 MANAGEMENT | No | Static/reserved infrastructure addresses |
| 20 USERS | Yes | 10.10.20.100ÃƒÂ¢Ã¢â€šÂ¬Ã¢â‚¬Å“10.10.20.199 |
| 30 DMZ | No | Static service addresses |
| 40 SECOPS | Yes | 10.10.40.100ÃƒÂ¢Ã¢â€šÂ¬Ã¢â‚¬Å“10.10.40.199 |
| 50 SERVERS | No | Static server addresses |
| 60 REDTEAM | Yes | 10.10.60.100ÃƒÂ¢Ã¢â€šÂ¬Ã¢â‚¬Å“10.10.60.199 |

Dnsmasq provides DHCP on USERS, SECOPS, and REDTEAM. Dnsmasq **Listen Port = 0** intentionally disables its DNS function. Unbound remains the primary recursive/caching DNS resolver on TCP/UDP 53.

The separate authoritative host/IP design includes static addresses outside the DHCP pools for `WIN11-01` (10.10.20.10) and `REDTEAM-01` (10.10.60.10).

## `LAB_NETWORKS` Alias

`LAB_NETWORKS` contains:

- 10.10.10.0/24
- 10.10.20.0/24
- 10.10.30.0/24
- 10.10.40.0/24
- 10.10.50.0/24
- 10.10.60.0/24

Description: **All Home Lab VLAN Networks**.

## Firewall Policy Baseline

OPNsense is stateful and rule order matters.

| Zone | Baseline intent |
|---|---|
| MANAGEMENT | DNS to firewall, HTTPS WebUI to firewall, block other firewall access, allow `LAB_NETWORKS`, allow outbound/Internet |
| USERS | DNS to firewall, block other firewall access, block `LAB_NETWORKS`, allow outbound/Internet |
| DMZ | DNS to firewall, block other firewall access, block `LAB_NETWORKS`, allow outbound/Internet |
| SECOPS | DNS to firewall, HTTPS WebUI to firewall, block other firewall access, allow `LAB_NETWORKS`, allow outbound/Internet |
| SERVERS | DNS to firewall, block other firewall access, block `LAB_NETWORKS`, allow outbound/Internet |
| REDTEAM | DNS to firewall, block other firewall access, block `LAB_NETWORKS`, allow outbound/Internet |

### MANAGEMENT ordered rules

1. PASS TCP/UDP ÃƒÂ¢Ã¢â€šÂ¬Ã¢â‚¬Â MANAGEMENT net ÃƒÂ¢Ã¢â‚¬Â Ã¢â‚¬â„¢ This Firewall ÃƒÂ¢Ã¢â€šÂ¬Ã¢â‚¬Â 53
2. PASS TCP ÃƒÂ¢Ã¢â€šÂ¬Ã¢â‚¬Â MANAGEMENT net ÃƒÂ¢Ã¢â‚¬Â Ã¢â‚¬â„¢ This Firewall ÃƒÂ¢Ã¢â€šÂ¬Ã¢â‚¬Â 443
3. BLOCK any ÃƒÂ¢Ã¢â€šÂ¬Ã¢â‚¬Â MANAGEMENT net ÃƒÂ¢Ã¢â‚¬Â Ã¢â‚¬â„¢ This Firewall
4. PASS any ÃƒÂ¢Ã¢â€šÂ¬Ã¢â‚¬Â MANAGEMENT net ÃƒÂ¢Ã¢â‚¬Â Ã¢â‚¬â„¢ `LAB_NETWORKS`
5. PASS any ÃƒÂ¢Ã¢â€šÂ¬Ã¢â‚¬Â MANAGEMENT net ÃƒÂ¢Ã¢â‚¬Â Ã¢â‚¬â„¢ any

### SECOPS ordered rules

1. PASS TCP/UDP ÃƒÂ¢Ã¢â€šÂ¬Ã¢â‚¬Â SECOPS net ÃƒÂ¢Ã¢â‚¬Â Ã¢â‚¬â„¢ This Firewall ÃƒÂ¢Ã¢â€šÂ¬Ã¢â‚¬Â 53
2. PASS TCP ÃƒÂ¢Ã¢â€šÂ¬Ã¢â‚¬Â SECOPS net ÃƒÂ¢Ã¢â‚¬Â Ã¢â‚¬â„¢ This Firewall ÃƒÂ¢Ã¢â€šÂ¬Ã¢â‚¬Â 443
3. BLOCK any ÃƒÂ¢Ã¢â€šÂ¬Ã¢â‚¬Â SECOPS net ÃƒÂ¢Ã¢â‚¬Â Ã¢â‚¬â„¢ This Firewall
4. PASS any ÃƒÂ¢Ã¢â€šÂ¬Ã¢â‚¬Â SECOPS net ÃƒÂ¢Ã¢â‚¬Â Ã¢â‚¬â„¢ `LAB_NETWORKS`
5. PASS any ÃƒÂ¢Ã¢â€šÂ¬Ã¢â‚¬Â SECOPS net ÃƒÂ¢Ã¢â‚¬Â Ã¢â‚¬â„¢ any

USERS, DMZ, SERVERS, and REDTEAM use the restricted-zone pattern described in the policy table above.

## Switch Integration

| Switch port | Role | VLAN membership |
|---|---|---|
| Port 1 | OPNsense trunk | Tagged 10/20/30/40/50/60 |
| Port 6 | `MGMT-BACKUP` | Untagged VLAN 10 |
| Port 7 | ENTHOST-01 physical attachment | Final Proxmox VLAN-aware guest-trunk details pending live validation |
| Port 8 | `MGMT-01` / MANAGEMENT PC access | Untagged VLAN 10 |
| Ports 2ÃƒÂ¢Ã¢â€šÂ¬Ã¢â‚¬Å“5, 9+ | TBD | Do not infer a permanent role |

`SW-Lab-01` management is 10.10.10.2/24 with gateway 10.10.10.1.

## Recovery History ÃƒÂ¢Ã¢â€šÂ¬Ã¢â‚¬Â Not Permanent Topology

- 192.168.99.0/24 was used as a temporary recovery LAN while VLAN 10 was repaired.
- Aruba port 7 temporarily carried VLAN 1 untagged during recovery; its current physical attachment is ENTHOST-01, with final Proxmox VLAN-aware guest-trunk details pending live validation.
- A temporary `PASS MANAGEMENT net ÃƒÂ¢Ã¢â‚¬Â Ã¢â‚¬â„¢ any` rule was used to avoid lockout during VLAN 10 activation. It should not remain as part of the final security baseline once permanent rules are validated.

## Evidence Status - Captured August 21, 2026

The sanitized OPNsense 26.7.2_2 live-configuration evidence set is stored under `screenshots/opnsense/`.

| Evidence | File |
|---|---|
| Interface assignments | [`01-opnsense-baseline-01-interface-assignments.png`](../screenshots/opnsense/01-opnsense-baseline-01-interface-assignments.png) |
| VLAN devices and parent/tag relationships | [`02-opnsense-baseline-02-vlan-devices.png`](../screenshots/opnsense/02-opnsense-baseline-02-vlan-devices.png) |
| Internal VLAN interface/gateway overview | [`03-opnsense-baseline-03-interface-overview.png`](../screenshots/opnsense/03-opnsense-baseline-03-interface-overview.png) |
| Dnsmasq DHCP configuration and pools | [`04-opnsense-baseline-04-dnsmasq-dhcp.png`](../screenshots/opnsense/04-opnsense-baseline-04-dnsmasq-dhcp.png) |
| Unbound DNS configuration | [`05-opnsense-baseline-05-unbound-dns.png`](../screenshots/opnsense/05-opnsense-baseline-05-unbound-dns.png) |
| `LAB_NETWORKS` firewall alias | [`06-opnsense-baseline-06-lab-networks-alias.png`](../screenshots/opnsense/06-opnsense-baseline-06-lab-networks-alias.png) |
| MANAGEMENT and SECOPS trusted-zone rules | [`07-opnsense-baseline-07-trusted-zone-rules.png`](../screenshots/opnsense/07-opnsense-baseline-07-trusted-zone-rules.png) |
| Representative REDTEAM restricted-zone rules | [`08-opnsense-baseline-08-restricted-zone-rules.png`](../screenshots/opnsense/08-opnsense-baseline-08-restricted-zone-rules.png) |

This evidence documents the configured interface/VLAN structure, DHCP/DNS split, reusable internal-network alias, and trusted/restricted firewall-policy patterns. End-to-end traffic-validation evidence remains separate under `screenshots/validation/` and must not be marked complete until the relevant endpoints and VLAN paths are tested.

Public evidence is sanitized to exclude public/ISP-assigned WAN details, complete MAC addresses, credentials, certificates/private keys, VPN peer secrets, and other sensitive identifiers.
## Source Basis

- `OPNsense-Phases-3-12-Home-Lab-Configuration-final-2026-08-20.pdf` ÃƒÂ¢Ã¢â€šÂ¬Ã¢â‚¬Â authoritative Phases 3ÃƒÂ¢Ã¢â€šÂ¬Ã¢â‚¬Å“12 baseline, direct ISP WAN, firewall policy, DHCP/DNS, recovery history, and switch integration.
- `Authoritative-VLAN-Designations-final-2026-08-20.txt` ÃƒÂ¢Ã¢â€šÂ¬Ã¢â‚¬Â VLAN networks and host/IP assignments.
- `Home-Lab-Network-Topology-final-2026-08-20.pdf` ÃƒÂ¢Ã¢â€šÂ¬Ã¢â‚¬Â direct ISP edge, Layer-3/Layer-2 roles, and workload placement.
- `Home-Lab-Hardware-Inventory-final-2026-08-20.pdf` ÃƒÂ¢Ã¢â€šÂ¬Ã¢â‚¬Â OPNsense hardware role and direct-ISP placement.
