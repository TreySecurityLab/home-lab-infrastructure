# Aruba Evidence

This directory stores sanitized evidence for the verified `SW-Lab-01` baseline.

## Captured August 20, 2026 — Pending Upload

The following evidence has been captured and should be uploaded here after sanitization review:

- switch identity / firmware state
- management IP 10.10.10.2/24 and default gateway 10.10.10.1
- port 1 OPNsense trunk tagged for VLANs 10/20/30/40/50/60
- port 6 `MGMT-BACKUP` VLAN 10 access state as applicable
- port 7 `ENTHOST-01` physical attachment
- port 8 `MGMT-01` VLAN 10 access state
- firmware transfer and upgraded firmware validation
- SNMP configuration before hardening
- successful SNMPv3 validation
- failed legacy SNMPv2c/public validation after hardening
- saved configuration / reboot persistence
- final time synchronization evidence (`13-time-sync.png`)

## Sanitization Requirements

Do not publish passwords, SNMP community strings, SNMPv3 authentication/privacy passwords, private keys, serial numbers, public WAN details, or unnecessary complete MAC addresses. Crop or redact unrelated identifiers before committing screenshots.

Use the Aruba AOS-S verification commands and evidence descriptions documented in `docs/03-aruba-switching.md` and `docs/08-validation-testing.md` where appropriate.
