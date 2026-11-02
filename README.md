# Zabbix Templates — SNMP Network, Printer and Infrastructure Monitoring

Open-source **Zabbix monitoring templates** maintained by [Kernovix](https://github.com/Kernovix). This repository provides importable YAML templates and practical setup documentation for monitoring printers, networking equipment and infrastructure with Zabbix.

Currently available: **Brother DCP-9020CDW SNMP printer monitoring for Zabbix 7.0** and **TP-Link Deco mesh monitoring for Zabbix 7.0**, covering node availability, controller telemetry and mesh-node discovery.

> **Independent community project.** Not affiliated with, sponsored by or endorsed by Zabbix SIA, Brother Industries or TP-Link. Templates are not official Zabbix integrations.

## Available Zabbix templates

| Category | Manufacturer and model | Protocol | Zabbix version | Documentation | Download |
| --- | --- | --- | --- | --- | --- |
| Printers | **Brother DCP-9020CDW** | SNMPv2c, ICMP | **7.0** | [Brother DCP-9020CDW monitoring guide](Printers/Brother/DCP-9020CDW/README.md) | [Zabbix 7.0 YAML template](Printers/Brother/DCP-9020CDW/7.0/template_brother_dcp9020cdw.yaml) |
| Network devices | **TP-Link Deco mesh systems** | ICMP, HTTPS, local encrypted API | **7.0** | [TP-Link Deco monitoring guide](Network_Devices/TP-Link/Deco/README.md) | [ICMP/HTTPS YAML](Network_Devices/TP-Link/Deco/7.0/template_tplink_deco_by_icmp_https.yaml) · [local-API YAML](Network_Devices/TP-Link/Deco/7.0/template_tplink_deco_by_local_api.yaml) |

### Brother DCP-9020CDW — SNMP printer monitoring

The [Brother DCP-9020CDW Zabbix template](Printers/Brother/DCP-9020CDW/README.md) is based on actual SNMP data collected from the printer. It includes:

- **Consumable monitoring:** CMYK toner readings using a Brother-specific private OID, plus black/cyan/magenta/yellow drum life and belt remaining life.
- **Printer monitoring:** uptime, availability, printer status, total page counter, input tray and error-state monitoring.
- **Network monitoring:** SNMP/ICMP reachability, interface discovery, reported link speed, inbound/outbound traffic, errors and discards.
- **Alerts and trends:** warning and critical thresholds, Zabbix triggers and graphs.

**Important limitations:** private toner percentages still require validation against the printer panel; signal strength (RSSI), SSID and wireless channel are **not available** from the SNMP data tested. The template has not yet been confirmed by a successful import into a live Zabbix 7.0 instance. Review the [tested OIDs and compatibility notes](Printers/Brother/DCP-9020CDW/OIDS.md).

### TP-Link Deco — mesh availability and controller telemetry

The [TP-Link Deco template package](Network_Devices/TP-Link/Deco/README.md) separates safe per-node monitoring from controller-only mesh telemetry:

- **ICMP/HTTPS template:** link it to every Deco, including satellites; no credentials are required.
- **Local-API template:** link it only to the controller; it collects CPU, memory, online-client count, mesh inventory, per-node status and reported link details through the local encrypted web API.
- **Alert safety:** controller API timeouts are handled as missing data, rather than false zero values or false offline nodes. The guide documents conservative confirmation windows and known firmware limitations.

**Important limitations:** the Deco local API is undocumented and may change between firmware versions. The local-API template requires a host-level Secret text password macro and must be validated on the target controller before alert actions are enabled. See the [Deco validation notes](Network_Devices/TP-Link/Deco/TESTING.md).

## Install a Zabbix template

1. Choose a device from the [template catalog](#available-zabbix-templates).
2. Read its model-specific documentation and download the YAML file.
3. In Zabbix, go to **Data collection → Templates → Import**.
4. Import the template and create a new monitored host.
5. Configure the correct SNMP host interface and its read-only credentials. Credentials are **not** embedded in the template.
6. Link the imported template to the host. Inspect **Monitoring → Latest data**, discovery results and unsupported items before configuring notifications.

For the Brother model, follow the full [installation and troubleshooting guide](Printers/Brother/DCP-9020CDW/README.md#installation). For TP-Link Deco, follow the [controller and satellite installation guide](Network_Devices/TP-Link/Deco/README.md#installation).

## Compatibility and project status

| Device | Model | Monitoring technology | Status |
| --- | --- | --- | --- |
| Brother | DCP-9020CDW | SNMPv2c / Printer-MIB / IF-MIB / ICMP | Community-developed; collected OIDs verified, Zabbix live import pending |
| TP-Link | Deco mesh systems | ICMP / HTTPS / local encrypted API | Community-developed; controller collection validated on Zabbix 7.0.32; see model/firmware and trigger limitations in the package guide |

This repository is intended to grow with additional Zabbix templates, organized by **category → manufacturer → model → Zabbix version**. Contributions are welcome, but untested templates will be clearly identified.

## Documentation and contributions

- [Brother DCP-9020CDW: monitoring, installation and FAQ](Printers/Brother/DCP-9020CDW/README.md)
- [Brother DCP-9020CDW: verified SNMP OID reference](Printers/Brother/DCP-9020CDW/OIDS.md)
- [TP-Link Deco: controller and satellite monitoring guide](Network_Devices/TP-Link/Deco/README.md)
- [TP-Link Deco: validation notes](Network_Devices/TP-Link/Deco/TESTING.md)
- [Contributing guidelines](CONTRIBUTING.md)
- [MIT license](LICENSE)

Please open a GitHub issue with reproducible behavior, Zabbix and firmware versions, sanitized examples, and the exact import error if applicable. Never share production SNMP community strings, tokens, private addresses, serial numbers or unsanitized SNMP walks.

## Frequently asked questions

**Does this repository provide a Brother DCP-9020CDW template for Zabbix 7.0?**  
Yes. The [YAML template](Printers/Brother/DCP-9020CDW/7.0/template_brother_dcp9020cdw.yaml) targets Zabbix 7.0 and includes printer, supplies and network metrics. It still needs validation in a live instance.

**Can Zabbix monitor Brother printer toner and drum levels over SNMP?**  
The standard Printer-MIB on this unit reports numerical drum and belt lifetime values. Its toner levels are not numerical percentages, so the model-specific template attempts to decode a private Brother maintenance field. Verify these values before using them for alerting.

**Does the template monitor Wi-Fi signal strength?**  
It monitors network-interface traffic and reported link status. RSSI, SSID and Wi-Fi channel were not exposed by the tested IEEE 802.11 SNMP MIB.

**Does this repository provide TP-Link Deco templates for Zabbix 7.0?**  
Yes. The [TP-Link Deco package](Network_Devices/TP-Link/Deco/README.md) contains an agentless ICMP/HTTPS template for every Deco and a separate local-API template for the mesh controller only. Review the documented API and firmware limitations before enabling alert actions.

---

Useful resources: [Zabbix documentation](https://www.zabbix.com/documentation/7.0/en) · [Zabbix community templates](https://github.com/zabbix/community-templates) · [RFC 3805 Printer-MIB](https://www.rfc-editor.org/rfc/rfc3805).
