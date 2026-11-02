# Contributing to Kernovix Zabbix Templates

Thanks for helping improve open-source Zabbix SNMP monitoring.

## Before opening a pull request

- Create a folder following `Category/Vendor/Model/ZabbixVersion/` for each new model and release.
- Provide a concise English-language `README.md` covering what the template monitors, supported hardware, prerequisites, installation, known limitations and references.
- Export the actual Zabbix template as YAML from the matching Zabbix version whenever possible.
- Do not reuse an existing UUID for unrelated items, discovery rules, triggers or graphs.
- Confirm each OID and data type with actual SNMP reads. Use the correct units and explain any vendor-specific conversions.
- For dynamic interfaces or consumables, prefer safe low-level discovery rules and avoid misleading alerts on disabled/unavailable interfaces.
- Test template import, item collection, preprocessing, trigger recovery and graph references. If any part has not been tested, say so explicitly.
- Keep all repository documentation, labels, YAML item names and trigger names **in English**.

## Reporting bugs

Please open an issue with:

1. Device manufacturer, exact model and firmware version.
2. Zabbix server/proxy release and SNMP version.
3. The affected item key, OID or trigger expression.
4. Exact import error or sanitized `Latest data` / unsupported item details.
5. Steps to reproduce, and whether the problem occurs with a real fault condition.

### Security and privacy

Before posting diagnostics, **remove SNMP communities, passwords, tokens, management IP addresses, MAC addresses, serial numbers and other device identifiers**. Never submit full, unreviewed SNMP walk dumps or production network inventory.

## Repository structure

```text
zabbix_templates/
├── README.md
├── CONTRIBUTING.md
├── LICENSE
└── Printers/
    └── Brother/
        └── DCP-9020CDW/
            ├── README.md
            ├── OIDS.md
            └── 7.0/
                └── template_brother_dcp9020cdw.yaml
```

## Project scope

This project publishes independent Zabbix monitoring templates. It is not an official Zabbix integration catalog. Avoid claiming support for models or firmware that have not been tested.

All contributions must be compatible with the repository's [MIT license](LICENSE).
