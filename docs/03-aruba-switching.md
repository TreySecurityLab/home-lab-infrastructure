# Aruba Switching

## Platform

| Item | Current baseline |
|---|---|
| Switch hostname | `SW-Lab-01` |
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

| VLAN | Name | Port 1 — OPNsense trunk | Port 8 — MANAGEMENT PC | Other ports |
|---:|---|---|---|---|
| 10 | MANAGEMENT | Tagged | Untagged | TBD |
| 20 | USERS | Tagged | — | TBD |
| 30 | DMZ | Tagged | — | TBD |
| 40 | SECOPS | Tagged | — | TBD |
| 50 | SERVERS | Tagged | — | TBD |
| 60 | REDTEAM | Tagged | — | TBD |

**Port 1** is the final OPNsense trunk and carries VLANs 10/20/30/40/50/60 as tagged traffic. There is no final untagged lab VLAN documented on this trunk.

**Port 8** is the final MANAGEMENT PC access port. The connected management system sends and receives ordinary untagged Ethernet; the switch places that traffic into VLAN 10.

**Ports 2–7 and 9+** remain TBD and must not be assigned permanent roles in the repository until the design is finalized.

### Historical Recovery Note

Port 7 temporarily carried VLAN 1 untagged while VLAN 10 connectivity was repaired. This was a recovery path only. It is not part of the current architecture and should remain listed as TBD.

## Verification Commands Supported by the Aruba Guide

| Command | Use in repository validation |
|---|---|
| `show vlan` | Lists VLAN information for the switch. |
| `show vlan <VLAN-ID>` | Shows the selected VLAN's name, VID, static/dynamic status, per-port mode (including tagged/untagged), and port up/down status. This is the primary evidence for port 1 and port 8 VLAN membership. |
| `show interfaces brief` | Shows current operating status for switch ports, useful for confirming the expected trunk/access links are up. |
| `show management` | Displays management-address information, including VLAN IP addressing and the default gateway. |
| `show running-config` | Displays the running configuration. Use a sanitized excerpt if configuration evidence is committed. |
| `show config` | Displays startup/saved configuration information. Useful for checking that the running VLAN/port state has been saved. |
| `show monitor` | Displays the configured mirror port and monitored sources if a local mirroring session is enabled. |

The Aruba guide warns that a mirroring exit port should connect only to a network analyzer, IDS, or other network-edge analysis device and not back into the production network. The planned Suricata/Zeek sensor path follows that separation principle.

## Current Evidence Targets

The OPNsense source document defines the following switch evidence as appropriate for the repository:

| Filename | Required visible elements | Status |
|---|---|---|
| `aruba-port1-opnsense-trunk.png` | Port 1 shown Tagged in VLANs 10–60 | Capture final clean evidence |
| `aruba-port8-management-access.png` | VLAN 10 shows port 1 Tagged and port 8 Untagged | Capture final clean evidence |

Additional useful evidence once available:

- VLAN inventory / per-VLAN membership views
- `show interfaces brief` with expected active links
- Sanitized startup/running configuration showing VLAN membership
- `show management` showing 10.10.10.2/24 and gateway 10.10.10.1
- `show monitor` after the passive security-sensor mirror path is implemented

## Configuration Persistence

After VLAN/port changes are finalized, the switch configuration should be saved so the VLAN membership and management configuration survive reboot. Repository evidence should distinguish the running state from the saved/startup configuration when both are captured.

## Sanitization

Do not publish switch passwords, SNMP community strings, serial numbers, complete MAC addresses, private keys, or other authentication material. The lab VLAN IDs/subnets, management IP 10.10.10.2, default gateway 10.10.10.1, and final port roles are architectural evidence and may remain visible.

## Source Basis

- *OPNsense Home Lab Configuration, Phases 3–12* — authoritative current switch-port baseline and management addressing.
- *Aruba 2530 Management and Configuration Guide for AOS-S 16.10* — J9774A applicability and supported VLAN, management, interface, mirroring, and configuration-verification commands.
- *Home Lab Network Topology Overview* — Layer-2 role and VLAN transport design.
