# OPNsense Firewall

## Documentation Status

**Hostname:** `opnsense-fw`  
**Platform:** Bare-metal OPNsense / 8 GB RAM  
**Documented configuration scope:** Phases 3–12 — interfaces, VLANs, DHCP, DNS, aliases, firewall policy, and Aruba switch integration.

This page separates the **current repository baseline** from temporary recovery settings used while VLAN 10 was repaired. Recovery values are retained only as implementation history so they are not mistaken for permanent architecture.

## Role in the Architecture

OPNsense is the Layer-3 gateway and policy-enforcement point for all six lab VLANs. It provides:

- WAN connectivity downstream of the Archer AXE75
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
| `re0` | WAN | IPv4 DHCP from Archer AXE75 |
| `ue0` | LAN/VLAN parent | Validated runtime parent toward `SW-Lab-01` port 1; carries VLAN tags 10/20/30/40/50/60 |

Earlier build notes referenced `ve0` or `vtnet1` during different hardware/interface-enumeration stages. The repository baseline uses the validated runtime interface name `ue0` unless the live firewall later reports a different actual interface.

### WAN Baseline

- Upstream router: 192.168.1.1
- WAN IPv4: DHCP from Archer AXE75
- The dynamic WAN lease should not be hard-coded into the public repository.
- Because the WAN intentionally sits behind an RFC1918 upstream network, **Block private networks** is unchecked so the Archer subnet is not discarded.
- Do not expose the OPNsense WebUI or SSH on WAN. Unsolicited WAN ingress should remain blocked unless a future lab intentionally introduces a tightly scoped published service/NAT rule.

## Basic System Configuration

| Setting | Documented value / action |
|---|---|
| Hostname | `opnsense-fw` |
| Domain | `lab.internal` |
| Timezone | `America/Chicago` |
| WAN IPv4 | DHCP from Archer AXE75 |
| Upstream router | 192.168.1.1 |

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

Each VLAN device is assigned as a normal OPNsense interface and enabled with the same baseline characteristics:

- IPv4 Configuration Type: **Static IPv4**
- IPv6 Configuration Type: **None**
- IPv4 Upstream Gateway: **None** for internal VLAN interfaces
- Block private networks: **unchecked** on internal VLANs
- Block bogon networks: **unchecked** on internal VLANs

| Interface | Enabled | IPv4 type | IPv6 type | Upstream gateway |
|---|---|---|---|---|
| MANAGEMENT | Yes | Static IPv4 | None | None |
| USERS | Yes | Static IPv4 | None | None |
| DMZ | Yes | Static IPv4 | None | None |
| SECOPS | Yes | Static IPv4 | None | None |
| SERVERS | Yes | Static IPv4 | None | None |
| REDTEAM | Yes | Static IPv4 | None | None |

## VLAN Gateways

| Interface | Static IPv4 address | Role |
|---|---|---|
| MANAGEMENT | 10.10.10.1/24 | Administrative gateway |
| USERS | 10.10.20.1/24 | User-zone gateway |
| DMZ | 10.10.30.1/24 | DMZ gateway |
| SECOPS | 10.10.40.1/24 | Security-operations gateway |
| SERVERS | 10.10.50.1/24 | Server-zone gateway |
| REDTEAM | 10.10.60.1/24 | Red-team gateway |

## DHCP Design

Dynamic addressing is enabled only for VLANs where endpoint churn is expected.

| VLAN | DHCP | Pool / addressing policy |
|---:|---|---|
| 10 MANAGEMENT | No | Static/reserved infrastructure addresses |
| 20 USERS | Yes | 10.10.20.100–10.10.20.199 |
| 30 DMZ | No | Static service addresses |
| 40 SECOPS | Yes | 10.10.40.100–10.10.40.199 |
| 50 SERVERS | No | Static server addresses |
| 60 REDTEAM | Yes | 10.10.60.100–10.10.60.199 |

### Dnsmasq / Unbound Split

Dnsmasq is used for DHCP on **USERS, SECOPS, and REDTEAM**. The documented design sets **Dnsmasq Listen Port = 0**, which disables the Dnsmasq DNS function so that Unbound can remain the primary recursive/caching resolver on DNS port 53.

For this baseline, port **53053 is not used**. It would only be appropriate after an intentional redesign of the DNS forwarding chain.

Clients use their VLAN `.1` address as the default gateway. DNS queries to OPNsense are permitted to **This Firewall** on TCP/UDP 53 by the Phase 12 firewall rules.

## `LAB_NETWORKS` Firewall Alias

| Field | Value |
|---|---|
| Enabled | Yes |
| Name | `LAB_NETWORKS` |
| Type | Network(s) |
| Description | All Home Lab VLAN Networks |

Contents:

- 10.10.10.0/24
- 10.10.20.0/24
- 10.10.30.0/24
- 10.10.40.0/24
- 10.10.50.0/24
- 10.10.60.0/24

`LAB_NETWORKS` is the canonical reusable alias for the baseline inter-VLAN rules.

## Firewall Policy Intent

| Interface | Policy intent |
|---|---|
| MANAGEMENT | Access all lab networks; administer OPNsense; Internet access |
| USERS | Internet access; no access to other lab networks; no OPNsense management |
| DMZ | Internet access; cannot initiate to other lab networks; no OPNsense management |
| SECOPS | Security/admin access to all lab networks; administer OPNsense; Internet access |
| SERVERS | Outbound Internet; cannot initiate to other VLANs; no OPNsense management except DNS |
| REDTEAM | Outbound Internet; isolated from other VLANs; no OPNsense management except DNS |

Rule order is significant. DNS exceptions are placed first, management exceptions follow where required, internal blocks/allows are applied next, and Internet/outbound passes are last.

## Ordered Firewall Rules

### MANAGEMENT

| # | Action | Protocol | Source | Destination | Port | Description |
|---:|---|---|---|---|---|---|
| 1 | PASS | TCP/UDP | MANAGEMENT net | This Firewall | 53 | Allow MANAGEMENT DNS to OPNsense |
| 2 | PASS | TCP | MANAGEMENT net | This Firewall | 443 | Allow MANAGEMENT WebUI to OPNsense |
| 3 | BLOCK | any | MANAGEMENT net | This Firewall | any | Block MANAGEMENT other access to OPNsense |
| 4 | PASS | any | MANAGEMENT net | `LAB_NETWORKS` | any | Allow MANAGEMENT to lab networks |
| 5 | PASS | any | MANAGEMENT net | any | any | Allow MANAGEMENT outbound / Internet |

### USERS

| # | Action | Protocol | Source | Destination | Port | Description |
|---:|---|---|---|---|---|---|
| 1 | PASS | TCP/UDP | USERS net | This Firewall | 53 | Allow USERS DNS to OPNsense |
| 2 | BLOCK | any | USERS net | This Firewall | any | Block USERS other access to OPNsense |
| 3 | BLOCK | any | USERS net | `LAB_NETWORKS` | any | Block USERS to lab networks |
| 4 | PASS | any | USERS net | any | any | Allow USERS outbound / Internet |

### DMZ

| # | Action | Protocol | Source | Destination | Port | Description |
|---:|---|---|---|---|---|---|
| 1 | PASS | TCP/UDP | DMZ net | This Firewall | 53 | Allow DMZ DNS to OPNsense |
| 2 | BLOCK | any | DMZ net | This Firewall | any | Block DMZ other access to OPNsense |
| 3 | BLOCK | any | DMZ net | `LAB_NETWORKS` | any | Block DMZ to lab networks |
| 4 | PASS | any | DMZ net | any | any | Allow DMZ outbound / Internet |

### SECOPS

| # | Action | Protocol | Source | Destination | Port | Description |
|---:|---|---|---|---|---|---|
| 1 | PASS | TCP/UDP | SECOPS net | This Firewall | 53 | Allow SECOPS DNS to OPNsense |
| 2 | PASS | TCP | SECOPS net | This Firewall | 443 | Allow SECOPS WebUI to OPNsense |
| 3 | BLOCK | any | SECOPS net | This Firewall | any | Block SECOPS other access to OPNsense |
| 4 | PASS | any | SECOPS net | `LAB_NETWORKS` | any | Allow SECOPS to lab networks |
| 5 | PASS | any | SECOPS net | any | any | Allow SECOPS outbound / Internet |

### SERVERS

| # | Action | Protocol | Source | Destination | Port | Description |
|---:|---|---|---|---|---|---|
| 1 | PASS | TCP/UDP | SERVERS net | This Firewall | 53 | Allow SERVERS DNS to OPNsense |
| 2 | BLOCK | any | SERVERS net | This Firewall | any | Block SERVERS other access to OPNsense |
| 3 | BLOCK | any | SERVERS net | `LAB_NETWORKS` | any | Block SERVERS to lab networks |
| 4 | PASS | any | SERVERS net | any | any | Allow SERVERS outbound / Internet |

### REDTEAM

| # | Action | Protocol | Source | Destination | Port | Description |
|---:|---|---|---|---|---|---|
| 1 | PASS | TCP/UDP | REDTEAM net | This Firewall | 53 | Allow REDTEAM DNS to OPNsense |
| 2 | BLOCK | any | REDTEAM net | This Firewall | any | Block REDTEAM other access to OPNsense |
| 3 | BLOCK | any | REDTEAM net | `LAB_NETWORKS` | any | Block REDTEAM to lab networks |
| 4 | PASS | any | REDTEAM net | any | any | Allow REDTEAM outbound / Internet |

## Switch Integration

| `SW-Lab-01` port | Final role | VLAN membership |
|---|---|---|
| Port 1 | OPNsense trunk | Tagged VLANs 10/20/30/40/50/60 |
| Port 8 | MANAGEMENT PC access | Untagged VLAN 10 |
| Ports 2–7, 9+ | TBD | No permanent role documented |

The final OPNsense trunk is documented as tagged-only for the six lab VLANs. No permanent untagged lab VLAN is defined on port 1.

## Recovery / Troubleshooting History

The following values were temporary and must **not** be presented as permanent topology:

- Bootstrap/recovery LAN 192.168.99.0/24
- OPNsense parent LAN 192.168.99.1/24 during recovery
- Management PC 192.168.99.10/24 during recovery
- `SW-Lab-01` port 7 carrying VLAN 1 untagged during recovery
- Temporary `PASS MANAGEMENT net -> any` migration rule

During the VLAN 10 issue, the configuration contained VLAN 10 but the interface did not initially exist in the running system. VLAN 10 and the MANAGEMENT assignment were rebuilt/reassigned while the recovery LAN remained available. Successful recovery was confirmed when MANAGEMENT appeared in Interfaces → Overview with 10.10.10.1/24.

The temporary MANAGEMENT allow-any rule was required above the broader MANAGEMENT block during troubleshooting. It should be removed/disabled after the permanent ordered rule set is validated.

## Current Validation Status

| Item | Status / repository treatment |
|---|---|
| VLAN IDs/names/subnets | Confirmed — publish |
| `.1` gateway convention | Confirmed design — publish |
| Port 1 OPNsense trunk | Confirmed current role — publish |
| Port 8 MANAGEMENT access | Confirmed current role — publish |
| Other switch ports | TBD — do not infer |
| VLAN 10 runtime activation | Recovered and confirmed during troubleshooting; recapture clean final evidence |
| VLAN 20/30/40/50/60 endpoint validation | Pending permanent port/host attachment |
| SPAN/mirror sensor port | Planned, not assigned in this baseline |

## Evidence Capture Plan

| Filename | Capture location | Required visible elements |
|---|---|---|
| `opnsense-phase03-interface-assignment.png` | Console / interface overview | WAN `re0` and LAN/trunk parent mapping |
| `opnsense-phase05-system-basics.png` | System settings | Hostname, domain, timezone |
| `opnsense-phase07-vlan-devices.png` | Interfaces → Devices → VLAN | All six VLAN tags/names and parent interface |
| `opnsense-phase08-interface-assignments.png` | Interfaces → Assignments | All six logical interfaces mapped to VLAN devices |
| `opnsense-phase09-interface-overview.png` | Interfaces → Overview | All six `.1/24` gateway addresses and interface status |
| `opnsense-phase10-dnsmasq-dhcp.png` | Services → Dnsmasq | Listen Port 0, DHCP interfaces, and all three pools |
| `opnsense-unbound-dns.png` | Services → Unbound DNS | Enabled resolver and intended interface settings |
| `opnsense-phase11-lab-networks-alias.png` | Firewall → Aliases | `LAB_NETWORKS` with all six subnets |
| `opnsense-phase12-management-rules.png` | Firewall → Rules → MANAGEMENT | Complete five-rule order |
| `opnsense-phase12-secops-rules.png` | Firewall → Rules → SECOPS | Complete five-rule order |
| `opnsense-phase12-isolated-zone-rules.png` | Firewall → Rules | Representative restricted-zone rules |

## Sanitization

Redact passwords, API keys, certificates/private keys, pre-shared keys, SNMP community strings, VPN peer secrets, recovery codes, complete MAC addresses, and public WAN details. Lab VLAN IDs/subnets, OPNsense `.1` gateways, `SW-Lab-01` management address 10.10.10.2, and final switch-port roles are part of the portfolio architecture and may remain visible.

## Source Basis

- *OPNsense Home Lab Configuration, Phases 3–12* — source of truth for this page.
- *Authoritative VLAN Designations* — fixed VLAN IDs, names, and networks.
- *Home Lab Network Topology Overview* — system placement and security-zone intent.
- *Home Lab Hardware Inventory* — firewall hardware role and RAM.
