# Brother DCP-9020CDW by SNMP

Community Zabbix 7.0 template for Brother DCP-9020CDW printers, developed using actual SNMP walk data collected from an NC-8100w firmware 1.16 device.

**[Download the Zabbix 7.0 YAML template](7.0/template_brother_dcp9020cdw.yaml)**

> Community template, not an official Zabbix or Brother integration. The YAML has not yet been confirmed by a live Zabbix 7.0 import. Validate each item and alert before relying on it in production.

## Features

- ICMP availability, round-trip time and packet loss
- SNMP availability, device identity, uptime and printer status
- Total printed page counter, tray capacity and printer alerts
- Automatic discovery of ten Printer-MIB supplies
- Belt and CMYK drum life remaining in percent
- Brother-specific private SNMP maintenance block used to derive CMYK toner levels
- Automatic IF-MIB network interface discovery, interface speed, inbound/outbound traffic, errors and discards
- Configurable warning and critical thresholds, graphs and triggers

## Installation

1. Download the [YAML](7.0/template_brother_dcp9020cdw.yaml).
2. In Zabbix 7.0 open **Data collection → Templates → Import** and import the file.
3. Create a host named, for example, `Brother DCP-9020CDW`.
4. Add an **SNMP interface** using the printer IP, UDP port **161**, **SNMPv2c**, and the device's read-only community.
5. Link **Brother DCP-9020CDW by SNMP** to the host.
6. Review **Monitoring → Latest data**, discovered supplies and interfaces, unsupported items and triggers.
7. Check the CMYK percentages against the printer web panel before enabling toner alert actions.

**Do not commit community strings, local IP addresses, serial numbers, or SNMP walk data containing sensitive inventory information.**

## Observed behavior and limitations

- The standard Printer-MIB returned `-3` for four toner cartridges and the waste box. It means an indeterminate amount remains, **not 0%**.
- The private maintenance OCTET STRING returned binary tags `0x81`–`0x84` with values interpreted as approximately black 50%, cyan 80%, magenta 80%, yellow 80%. These values require confirmation against the printer UI and checking how Zabbix represents the raw data.
- The belt and drum supply entries provide numeric maximum and remaining values, allowing reliable percent calculations.
- Four IF-MIB interfaces were exposed: index 1 active at a reported 72 Mbps, index 2 inactive at 100 Mbps, index 3 inactive at 72 Mbps and index 4 a loopback. Do not infer Wi-Fi versus Ethernet solely from `ifType`.
- No data was returned from the tested IEEE 802.11 MIB. **RSSI, SSID, channel and signal quality are not available** through those OIDs.
- The input tray level may be `-3` and cannot be turned into a percent.
- Paper jam, cover open and other fault triggers rely on standard error mappings but must be tested against real printer events to exclude false positives.
- An inactive wired or wireless interface is normal when another is in use. Link-down alerts should be enabled only for selected interfaces.

## Monitoring thresholds

| Setting | Default |
| --- | --- |
| Critical consumable level | 10% |
| Warning consumable level | 20% |
| Critical recovery | 12% |
| Warning recovery | 22% |
| Interface link-down monitoring | Disabled by default |
| ICMP high packet loss | 20% |
| ICMP high latency | 150 ms |

Change host or template macros to suit your environment. Avoid duplicate host alert actions for related warning and critical conditions.

## Troubleshooting

If SNMP items are unsupported, check UDP 161, SNMPv2c community, and the host interface configuration. If toner values are unsupported, use the raw private maintenance item's actual value in Zabbix's preprocessing test and verify its decoding. If discovery fails, test IF-MIB `1.3.6.1.2.1.2.2.1.2` and Printer-MIB `1.3.6.1.2.1.43.11.1.1.6` using `snmpwalk`.

For the verified OID inventory, see [OIDS.md](OIDS.md).

## References

- [Zabbix community templates](https://github.com/zabbix/community-templates)
- [Zabbix 7.0 template import/export documentation](https://www.zabbix.com/documentation/7.0/en/manual/xml_export_import/templates)
- [RFC 3805 Printer-MIB](https://www.rfc-editor.org/rfc/rfc3805)
- [RFC 2790 Host Resources MIB](https://www.rfc-editor.org/rfc/rfc2790)
- [RFC 2863 IF-MIB](https://www.rfc-editor.org/rfc/rfc2863)

## License

MIT; see the [repository LICENSE](../../../../LICENSE).
