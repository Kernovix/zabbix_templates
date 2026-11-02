# Zabbix Templates

Community-maintained Zabbix monitoring templates by **Kernovix**.

> Independent project. Not affiliated with or endorsed by Zabbix SIA or Brother Industries.

## Template catalog

| Category | Vendor | Device | Supported Zabbix release | Documentation | Template |
| --- | --- | --- | --- | --- | --- |
| Printers | Brother | DCP-9020CDW | 7.0 | [Details](Printers/Brother/DCP-9020CDW/README.md) | [YAML](Printers/Brother/DCP-9020CDW/7.0/template_brother_dcp9020cdw.yaml) |

## Getting started

Read the device-specific README and import its YAML from **Data collection → Templates → Import**. Create a host with the correct SNMP interface and credentials; templates do not contain credentials or private IP addresses.

## Support and validation

These templates are community-maintained and are not official Zabbix integrations. The Brother template was developed against actual SNMP walk results; **successful import into a live Zabbix 7.0 server and all device alerts have not yet been verified**. See the individual documentation for supported OIDs, assumptions, limitations and troubleshooting.

## Contributing

Issues and pull requests are welcome. Please provide the Zabbix release, printer model and firmware, exact error messages, and sanitized SNMP examples. **Never publish SNMP credentials, private IPs, MAC addresses, serial numbers or other sensitive inventory data.**

## License

MIT — see [LICENSE](LICENSE).
