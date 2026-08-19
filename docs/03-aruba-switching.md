# Aruba Switching

## Platform

| Item | Current baseline |
|---|---|
| Switch hostname | `SW-Lab-01` |
| Inventory label | `SW-LAB-01` |
| Model | Aruba J9774A |
| Operating family | ArubaOS-Switch / AOS-S |
| Role | Managed Layer-2 switching, VLAN segmentation, 802.1Q tagging/trunking, access-port assignment, and traffic mirroring |
| Management network | VLAN 10 — MANAGEMENT |
| Management address | 10.10.10.2/24 |
| Default gateway | 10.10.10.1 (`opnsense-fw`) |

The Aruba 2530 Management and Configuration Guide for AOS-S 16.10 explicitly lists J9774A as an applicable Aruba 2530 model.

## Layer-2 Role

`SW-Lab-01` does not provide inter-VLAN routing in the documented baseline. OPNsense is the Layer-3 gateway and firewall boundary. The switch provides Layer-2 segmentation and transports the six VLANs between OPNsense and VLAN-aware systems.

## Authoritative VLAN Membership

| VLAN | Name | Port 1 — OPNsense trunk | Port 8 — `MGMT-01` access | Other ports |
|---:|---|---|---|---|
| 10 | MANAGEMENT | Tagged | Untagged | TBD |
| 20 | USERS | Tagged | — | TBD |
| 30 | DMZ | Tagged | — | TBD |
| 40 | SECOPS | Tagged | — | TBD |
| 50 | SERVERS | Tagged | — | TBD |
| 60 | REDTEAM | Tagged | — | TBD |

**Port 1** is the final OPNsense trunk and carries VLANs 10/20/30/40/50/60 as tagged traffic. There is no final untagged lab VLAN documented on this trunk.

**Port 8** is the final MANAGEMENT PC access port. The authoritative host/IP source identifies that management system as `MGMT-01` at 10.10.10.6. The endpoint sends/receives ordinary untagged Ethernet and the switch places that traffic into VLAN 10.

**Ports 2–7 and 9+** remain TBD and must not be assigned permanent roles until explicitly finalized.

### Historical Recovery Note

Port 7 temporarily carried VLAN 1 untagged while VLAN 10 connectivity was repaired. This was a recovery path only. It is not part of the current architecture and remains TBD.

## Verification Commands Supported by the Aruba Guide

| Command | Use in repository validation |
|---|---|
| `show vlan` | Lists VLAN information for the switch |
| `show vlan <VLAN-ID>` | Shows the selected VLAN's name, VID, status, per-port mode including tagged/untagged membership, and port state |
| `show interfaces brief` | Shows current operating status for switch ports |
| `show management` | Displays management-address information, VLAN IP addressing, and the default gateway |
| `show running-config` | Displays the running configuration; publish only sanitized excerpts |
| `show config` | Displays startup/saved configuration information and helps confirm persistence |
| `show monitor` | Displays configured mirror destination/source information when local mirroring is enabled |

The Aruba guide identifies time synchronization, SNMPv3, port status, VLAN information, local mirroring, and configuration verification as supported management capabilities for the 2530 family. These can be documented as they are actually deployed and verified.

## Current Evidence Targets

| Filename | Required visible elements | Status |
|---|---|---|
| `aruba-port1-opnsense-trunk.png` | Port 1 shown Tagged in VLANs 10–60 | Capture final clean evidence |
| `aruba-port8-management-access.png` | VLAN 10 shows port 1 Tagged and port 8 Untagged | Capture final clean evidence |
| `aruba-management-address.png` | Management IP 10.10.10.2/24 and gateway 10.10.10.1 | Capture final clean evidence |

Additional useful evidence once available:

- VLAN inventory / per-VLAN membership views
- `show interfaces brief` with expected active links
- Sanitized startup/running configuration showing VLAN membership
- `show management` showing 10.10.10.2/24 and gateway 10.10.10.1
- `show ntp status` or equivalent after time synchronization is finalized
- SNMPv3 status after SNMPv3 is implemented
- `show monitor` after the passive security-sensor mirror path is implemented

## Configuration Persistence

After VLAN/port changes are finalized, save the switch configuration so VLAN membership and management settings survive reboot. Repository evidence should distinguish the running state from the saved/startup configuration when both are captured.

## Sanitization

Do not publish switch passwords, SNMP community strings, serial numbers, complete MAC addresses, private keys, or other authentication material. The lab VLAN IDs/subnets, management IP 10.10.10.2, default gateway 10.10.10.1, and final port roles are architectural evidence and may remain visible.

## Source Basis

- *OPNsense Home Lab Configuration — UPDATED* — authoritative current switch-port baseline and management addressing.
- *Aruba 2530 Management and Configuration Guide for AOS-S 16.10* — J9774A applicability and supported VLAN, management, interface, NTP/SNTP, SNMPv3, mirroring, and configuration-verification capabilities.
- *Authoritative VLAN Designations — final* — `SW-LAB-01` and `MGMT-01` addressing.
- *Home Lab Network Topology Overview — UPDATED* — Layer-2 role and VLAN transport design.
