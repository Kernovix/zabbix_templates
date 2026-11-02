# Brother DCP-9020CDW Zabbix 7.0 SNMP Monitoring Template

**Community Zabbix 7.0 template for the Brother DCP-9020CDW color laser multifunction printer.** Monitor CMYK toner levels, drum and belt lifetime, printed pages, printer errors, input tray, SNMP availability and Ethernet/Wi-Fi network traffic.

**[Download Brother DCP-9020CDW Zabbix YAML template](7.0/template_brother_dcp9020cdw.yaml)** · [SNMP OID reference](OIDS.md) · [All Kernovix Zabbix templates](../../../../README.md)

> **Status: community-developed / live Zabbix import pending.** Based on actual SNMP walk results from a Brother DCP-9020CDW with NC-8100w firmware 1.16. Not an official Brother or Zabbix template. Validate the import, preprocessing and alert behavior in your instance before depending on notifications.

## Features: toner, drums, pages, printer status and network

| Monitoring area | Included measurements | Notes |
| --- | --- | --- |
| Toner cartridges | Black, cyan, magenta, yellow | Brother private maintenance OID; percentages require panel validation |
| Drums | Black, cyan, magenta, yellow | Remaining percentage derived from Printer-MIB numeric capacities |
| Belt | Remaining life percentage | Printer-MIB |
| Waste toner box | Raw supply state | Numeric remaining percentage not exposed in tested standard MIB |
| Printer | Online status, uptime, model, firmware, print page counter | SNMP and ICMP |
| Paper and alerts | Input tray information, internal printer status, error state | Actual fault codes should be tested |
| Wired and wireless network | IF-MIB interface discovery, link state, link speed, traffic, errors and discards | No SSID, RSSI or channel discovery |
| Zabbix alerting | Supply thresholds, offline monitoring, printer errors and network alerts | Enable optional link-down alerts only for interfaces intended to be active |

## Requirements and compatibility

- **Zabbix:** template exported for **7.0**; live import testing still pending.
- **Printer:** Brother DCP-9020CDW. Tested SNMP OIDs were read from NC-8100w firmware 1.16.
- **Protocol:** SNMPv2c on UDP/161 and ICMP reachable from the Zabbix server or proxy.
- **Permissions:** SNMP read-only community is sufficient. Keep credentials outside this repository.
- **Other Brother printers:** not validated. Private OIDs and consumable indexes may differ.

## Installation

1. Download the [Brother DCP-9020CDW template YAML](7.0/template_brother_dcp9020cdw.yaml).
2. In **Zabbix 7.0**, open **Data collection → Templates → Import** and select the YAML.
3. Confirm the imported template is named `Brother DCP-9020CDW by SNMP`.
4. Create a host under **Data collection → Hosts → Create host**.
5. Add a default **SNMP** interface with the printer's IP address, port **161**, SNMP version **v2c** and the printer's read-only community.
6. Link the imported template to the host. No Zabbix agent is required on the printer.
7. Check **Monitoring → Latest data**, **Discovery** and the host's unsupported items.
8. Compare black, cyan, magenta and yellow toner percentages with the printer's own control panel before trusting toner-related triggers.

To verify basic SNMP access from a Linux host:

```bash
snmpget -v2c -c 'YOUR_READ_ONLY_COMMUNITY' PRINTER_IP 1.3.6.1.2.1.1.1.0
snmpwalk -v2c -c 'YOUR_READ_ONLY_COMMUNITY' PRINTER_IP 1.3.6.1.2.1.43.11.1.1.6
snmpwalk -v2c -c 'YOUR_READ_ONLY_COMMUNITY' PRINTER_IP 1.3.6.1.2.1.2.2.1.2
```

The second command lists printer supply descriptions; the third lists IF-MIB interface descriptions. Do not publish unsanitized output containing device identifiers.

## SNMP monitoring details

### Brother toner and printer supplies

The device returned ten Printer-MIB supply entries: black/cyan/magenta/yellow toner cartridges, waste toner box, belt unit and four drums.

For the four toner cartridges, `prtMarkerSuppliesLevel` returned **-3** and `prtMarkerSuppliesMaxCapacity` returned **-2**. These indicate nonquantified states, **not percentages**. The template includes Brother's private maintenance OCTET STRING `1.3.6.1.4.1.2435.2.3.9.4.2.1.5.5.8.0` to attempt CMYK percentages. The initial captured TLV sample suggests approximately **50% black and 80% each cyan, magenta and yellow**, but these figures have **not been cross-checked against the printer display**. Zabbix representation of OCTET STRING data must also be checked.

For drums and the belt, the Printer-MIB provided numeric remaining values and maximum capacity. The percentage calculation uses **remaining / maximum × 100**.

### Ethernet and Wi-Fi interface monitoring

The IF-MIB showed four interfaces:

| SNMP interface index | Description | Observed operational state | Reported interface speed |
| --- | --- | --- | --- |
| 1 | `NC-8100w` | Up | 72 Mbps |
| 2 | `NC-8500h` | Down | 100 Mbps |
| 3 | `NC-8100w` | Down | 72 Mbps |
| 4 | `SoftwareLoopBack` | Up | 0 bps |

The first interface is likely the active Wi-Fi connection, but the interface table alone does not conclusively identify the physical medium: the firmware reports `ifType=7` for three interfaces. **72 Mbps is a reported link/interface speed, not measured internet bandwidth**.

The IEEE 802.11 OID tree `1.2.840.10036` did not provide usable data in testing. Therefore **Wi-Fi signal strength (RSSI), SSID, wireless channel and signal quality are not monitored**. An unused Ethernet or Wi-Fi interface may legitimately remain down.

### Alerts, thresholds and graphs

| Alert or threshold | Default design |
| --- | --- |
| Device unavailable | 5 minutes |
| SNMP polling unavailable | 10 minutes |
| Critical supply level | Below 10% |
| Warning supply level | Below 20% |
| Critical recovery | Above 12% |
| Warning recovery | Above 22% |
| Paper jam / out of paper / cover open | Based on error MIB codes; validate against real events |
| Network interface down | Optional; disabled by default |

Warnings on non-numeric supply values (`-2`, `-3`) must not be treated as zero toner. Inspect the discovered trigger prototypes, graphs and latest data before configuring actions.

## Troubleshooting

**Zabbix reports `Timeout` or SNMP items are unsupported.** Confirm that SNMP is enabled, the host is reachable from the *Zabbix server or proxy* over UDP/161, the SNMP community matches and the version is v2c.

**Supply discovery fails.** Query the standard Printer-MIB supply description column: `1.3.6.1.2.1.43.11.1.1.6`. Review discovery errors and unsupported preprocessing steps.

**Toner percentages look wrong or unsupported.** Inspect the raw private-maintenance item and use Zabbix preprocessing tests to validate binary/hex representation. Confirm the percentage against the printer's web interface before enabling alert actions.

**Some network interfaces show Down.** Compare operational and administrative states before enabling interface-down alerts. A second inactive network interface is not necessarily a fault.

**Paper or cover alert seems incorrect.** Printer errors can be vendor/firmware-specific. Report a sanitized sample of the actual error OID while the fault is present.

See [OIDS.md](OIDS.md) for the full list of known OIDs and [the repository home](../../../../README.md) for other Zabbix templates.

## Frequently asked questions (FAQ)

### Is there a Zabbix 7.0 template for the Brother DCP-9020CDW?

Yes. This repository contains a [Zabbix 7.0 YAML template](7.0/template_brother_dcp9020cdw.yaml) designed specifically around observed SNMP values from the Brother DCP-9020CDW. Import and live trigger behavior still require verification.

### Can Zabbix monitor Brother DCP-9020CDW toner levels?

Potentially. On this device, the standard Printer-MIB only reports indeterminate toner states. A Brother-specific maintenance OID appears to expose encoded CMYK levels. Check against the printer's displayed levels before using percentage-based alerts.

### Can Zabbix monitor Brother printer drum and belt life?

Yes, the tested Printer-MIB returns numeric values for the four drums and belt, allowing remaining-life percentages to be calculated.

### Does Zabbix show Brother Wi-Fi RSSI or the connected SSID?

Not with the tested OIDs. The IF-MIB exposes traffic, status and interface speed, but the tested IEEE 802.11 tree did not return SSID, channel or RSSI data.

### Does this Zabbix template support other Brother MFC and DCP models?

Compatibility has not been validated on other models. SNMP table indexes, private OIDs and device error mappings may change across firmware families.

## References and license

- [SNMP OID reference specific to DCP-9020CDW](OIDS.md)
- [Zabbix 7.0 template import/export documentation](https://www.zabbix.com/documentation/7.0/en/manual/xml_export_import/templates)
- [Zabbix community templates](https://github.com/zabbix/community-templates)
- [Printer-MIB (RFC 3805)](https://www.rfc-editor.org/rfc/rfc3805)
- [IF-MIB (RFC 2863)](https://www.rfc-editor.org/rfc/rfc2863)
- [Host Resources MIB (RFC 2790)](https://www.rfc-editor.org/rfc/rfc2790)

Community-maintained under the [MIT license](../../../../LICENSE). Contributions and reproducible bug reports are welcome via [GitHub issues](https://github.com/Kernovix/zabbix_templates/issues).
