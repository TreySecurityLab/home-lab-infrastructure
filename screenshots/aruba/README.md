# Aruba Evidence

Sanitized evidence for the verified `SW-Lab-01` management, VLAN, firmware, SNMPv3, and time-synchronization baseline.

## Final Evidence Set - August 20, 2026

1. `01-switch-baseline-01-trunk-vlans.png` - port 1 membership across VLANs 10/20/30/40/50/60
2. `02-switch-baseline-02-vlan10-membership.png` - VLAN 10 membership for ports 1, 6, 7, and 8
3. `03-switch-baseline-03-management-ip-gateway.png` - management IP 10.10.10.2/24 and gateway 10.10.10.1
4. `04-switch-baseline-04-firmware-before.png` - pre-upgrade firmware YA.16.06.0006
5. `05-switch-baseline-05-firmware-after.png` - post-upgrade firmware YA.16.10.0010
6. `06-switch-baseline-06-snmpv3-success.png` - successful SNMPv3 monitoring validation
7. `07-switch-baseline-07-public-community-removed.png` - legacy public SNMP community removed
8. `08-switch-baseline-08-sntp-status.png` - SNTP configuration/status and current switch time

## Sanitization

Do not publish passwords, SNMP community strings, SNMPv3 authentication/privacy passwords, private keys, serial numbers, public WAN information, unnecessary complete MAC addresses, or unnecessary personal identifiers.

The firmware evidence was sanitized before publication to obscure personal contact/location information, the switch base MAC address, and serial number.
