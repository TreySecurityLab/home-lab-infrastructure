# Aruba Switching

## Platform

| Item | Current baseline |
|---|---|
| Switch hostname | `SW-Lab-01` |
| Inventory label | `SW-LAB-01` |
| Model | Aruba J9774A |
| Operating family | ArubaOS-Switch / AOS-S |
| Current firmware | `YA.16.10.0010` |
| Role | Managed Layer-2 switching, VLAN segmentation, 802.1Q tagging/trunking, access-port assignment, and traffic mirroring |
| Management network | VLAN 10 — MANAGEMENT |
| Management address | 10.10.10.2/24 |
| Default gateway | 10.10.10.1 (`opnsense-fw`) |

The Aruba 2530 Management and Configuration Guide for AOS-S 16.10 explicitly lists J9774A as an applicable Aruba 2530 model.

## Layer-2 Role

`SW-Lab-01` does not provide inter-VLAN routing in the documented baseline. OPNsense is the Layer-3 gateway and firewall boundary. The switch provides Layer-2 segmentation and transports the six VLANs between OPNsense and VLAN-aware systems.

## Authoritative Port Assignments

| Port | Current role | VLAN / tagging state |
|---:|---|---|
| 1 | OPNsense trunk | Tagged VLANs 10/20/30/40/50/60 |
| 6 | `MGMT-BACKUP` laptop | Untagged VLAN 10 MANAGEMENT |
| 7 | `ENTHOST-01` physical attachment | Permanent host attachment established; final Proxmox VLAN-aware guest trunk/tagging details remain pending live validation |
| 8 | `MGMT-01` laptop | Untagged VLAN 10 MANAGEMENT |
| 2–5, 9+ | TBD | No permanent role should be inferred |

**Port 1** is the final OPNsense trunk and carries VLANs 10/20/30/40/50/60 as tagged traffic. There is no final untagged lab VLAN documented on this trunk.

**Port 6** is the backup-management access port. `MGMT-BACKUP` was validated on VLAN 10 with local address 10.10.10.10/24 while physically connected.

**Port 7** is now the permanent physical attachment for `ENTHOST-01`. The prior temporary recovery use of this port is historical only. Final VLAN-aware guest tagging for VLANs 20/30/50 will be documented after the Proxmox networking work is completed and verified.

**Port 8** is the primary MANAGEMENT access port for `MGMT-01` at 10.10.10.6.

## Completed Management, Firmware, and Security Baseline — August 20, 2026

The switch-baseline workflow was completed in the required order: management verification, backup, firmware upgrade, post-upgrade validation, SNMPv3 deployment, legacy-SNMP rejection testing, configuration save, reboot, and persistence validation.

### Management and Configuration

- Management address verified at 10.10.10.2/24 on VLAN 10.
- Default gateway verified at 10.10.10.1.
- Port 1 remained the tagged OPNsense trunk for VLANs 10/20/30/40/50/60.
- Port 6 is untagged VLAN 10 for `MGMT-BACKUP`.
- Port 7 is assigned to `ENTHOST-01`.
- Port 8 is untagged VLAN 10 for `MGMT-01`.
- Running configuration was saved to persistent configuration.
- An off-switch configuration backup was exported and retained for recovery/reference.

### Firmware

- The Aruba 2530/J9774A firmware image `YA_16_10_0010.swi` was validated on the TFTP host before transfer.
- Firmware was transferred using the Aruba-supported TFTP workflow.
- The switch rebooted successfully into `YA.16.10.0010`.
- Management addressing and VLAN configuration remained intact after the firmware reboot.

### Time Synchronization

- Switch time synchronization was configured and verified.
- Time-sync state was rechecked as part of the final post-reboot evidence set.
- Exact upstream time-server details are not published in the repository unless needed for sanitized evidence.

### SNMPv3 Hardening

- SNMPv3 was enabled only after firmware and management validation succeeded.
- A dedicated authenticated monitoring user was configured for SNMPv3 monitoring.
- SNMPv3 monitoring was tested successfully from the management environment.
- Legacy SNMPv1/v2c access using the former public community was then disabled/restricted from use.
- Positive/negative validation was completed: SNMPv3 succeeded while the legacy SNMPv2c test failed as intended.
- Authentication/privacy credentials and community strings are intentionally omitted from repository documentation and screenshots.

### Persistence Validation

After the final configuration save and reboot, the switch was checked for persistence of the management address, gateway, VLAN state, firmware, SNMPv3 hardening, and time synchronization. The completed configuration survived reboot.

## Verification Commands Supported by the Aruba Guide

| Command | Use in repository validation |
|---|---|
| `show vlan` | Lists VLAN information for the switch |
| `show vlan <VLAN-ID>` | Shows the selected VLAN's name, VID, status, per-port mode including tagged/untagged membership, and port state |
| `show interfaces brief` | Shows current operating status for switch ports |
| `show management` | Displays management-address information, VLAN IP addressing, and the default gateway |
| `show running-config` | Displays the running configuration; publish only sanitized excerpts |
| `show config` | Displays startup/saved configuration information and helps confirm persistence |
| `show snmp-server` | Displays SNMP configuration state; sanitize community/authentication information before publication |
| `show monitor` | Displays configured mirror destination/source information when local mirroring is enabled |

The Aruba guide identifies time synchronization, SNMPv3, port status, VLAN information, local mirroring, and configuration verification as supported management capabilities for the 2530 family.

## Evidence Status

The final switch evidence was captured on August 20, 2026 and is pending sanitized upload to `screenshots/aruba/`.

| Evidence category | Current status |
|---|---|
| Switch identity / firmware baseline | Captured; pending upload |
| Management IP and default gateway | Captured; pending upload |
| Port 1 tagged OPNsense trunk | Captured; pending upload |
| Port 6 `MGMT-BACKUP` VLAN 10 access | Captured/previously validated; pending repository evidence placement as applicable |
| Port 7 `ENTHOST-01` attachment | Captured in current switch state; pending upload |
| Port 8 `MGMT-01` VLAN 10 access | Captured; pending upload |
| Firmware transfer / post-upgrade firmware | Captured; pending upload |
| SNMP state before hardening | Captured; pending upload |
| SNMPv3 successful validation | Captured; pending upload |
| Legacy SNMPv2c failure after hardening | Captured; pending upload |
| Saved configuration / reboot persistence | Captured; pending upload |
| Time synchronization | Captured as `13-time-sync.png`; pending upload |

## Historical Recovery Note

Port 7 temporarily carried recovery connectivity while VLAN 10 was repaired. That recovery role is no longer current. Port 7 is now assigned to `ENTHOST-01`. The historical 192.168.99.0/24 recovery network is not part of the permanent architecture.

## Future Switching Work

The following items remain pending and must not be presented as complete:

- Final `ENTHOST-01` VLAN-aware Proxmox trunk/tagging validation on port 7
- Final `SECHOST-01` physical port assignment and VLAN 40 transport
- Permanent SPAN/mirror destination port and passive NSM sensor path
- Final unused-port shutdown/administrative-disable policy

## Sanitization

Do not publish switch passwords, SNMP community strings, SNMPv3 authentication/privacy passwords, serial numbers, complete MAC addresses, private keys, or other authentication material. The lab VLAN IDs/subnets, management IP 10.10.10.2, default gateway 10.10.10.1, firmware version, and final port roles are architectural evidence and may remain visible.

## Source Basis

- *OPNsense Home Lab Configuration — UPDATED* — management addressing and OPNsense trunk baseline.
- *Aruba 2530 Management and Configuration Guide for AOS-S 16.10* — J9774A applicability and supported VLAN, management, interface, time-sync, SNMPv3, mirroring, firmware, and configuration-verification capabilities.
- *Authoritative VLAN Designations — final* — management and workload addressing baseline.
- *Home Lab Network Topology Overview — UPDATED* — Layer-2 role and VLAN transport design.
- *MGMT-01 Remote RDP Resolution Summary* — verified port 6 and port 8 VLAN 10 access behavior.
- Live SW-Lab-01 configuration and validation evidence captured August 20, 2026 — firmware, SNMPv3, time synchronization, configuration persistence, and current physical port assignments.
