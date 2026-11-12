# Zabbix Templates — Network, Printer and Infrastructure Monitoring

Open-source **Zabbix monitoring templates** maintained by [Kernovix](https://github.com/Kernovix). This repository provides importable YAML templates and practical setup documentation for monitoring printers, networking equipment and infrastructure using SNMP, HTTP APIs, ICMP and device-native metrics.

Currently available: **Brother DCP-9020CDW printer monitoring**, **TP-Link Deco mesh monitoring**, **SMLIGHT SLZB-MR4U monitoring** and **TrueNAS SCALE SNMP/ZFS ARC-aware monitoring**, all for Zabbix 7.0.

> **Independent community project.** Not affiliated with, sponsored by or endorsed by Zabbix SIA, Brother Industries, TP-Link or SMLIGHT. Templates are not official Zabbix integrations.

## Available Zabbix templates

| Category | Manufacturer and model | Protocol | Zabbix version | Documentation | Download |
| --- | --- | --- | --- | --- | --- |
| Printers | **Brother DCP-9020CDW** | SNMPv2c, ICMP | **7.0** | [Brother DCP-9020CDW monitoring guide](Printers/Brother/DCP-9020CDW/README.md) | [Zabbix 7.0 YAML template](Printers/Brother/DCP-9020CDW/7.0/template_brother_dcp9020cdw.yaml) |
| Network devices | **TP-Link Deco mesh systems** | ICMP, HTTPS, local encrypted API | **7.0** | [TP-Link Deco monitoring guide](Network_Devices/TP-Link/Deco/README.md) | [ICMP/HTTPS YAML](Network_Devices/TP-Link/Deco/7.0/template_tplink_deco_by_icmp_https.yaml) · [local-API YAML](Network_Devices/TP-Link/Deco/7.0/template_tplink_deco_by_local_api.yaml) |
| Network devices | **SMLIGHT SLZB-MR4U** | HTTP /metrics | **7.0** | [SMLIGHT SLZB-MR4U monitoring guide](Network_Devices/SMLIGHT/SLZB-MR4U/README.md) | [Zabbix 7.0 YAML template](Network_Devices/SMLIGHT/SLZB-MR4U/7.0/template_smlight_slzb_mr4u_by_http.yaml) |
| Storage devices | **TrueNAS SCALE (ZFS ARC-aware memory)** | SNMP, ICMP | **7.0** | [TrueNAS SCALE monitoring guide](Storage_Devices/TrueNAS/template_truenas_scale_by_snmp/README.md) | [Zabbix 7.0 YAML](Storage_Devices/TrueNAS/template_truenas_scale_by_snmp/7.0/template_truenas_scale_by_snmp.yaml) |

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
- **Local-API template:** link it only to the controller; it collects CPU, memory, client totals, mesh inventory, per-node status and reported link details through the local encrypted web API.
- **Alert safety:** controller API timeouts are handled as missing data, rather than false zero values or false offline nodes. The guide documents conservative confirmation windows and known firmware limitations.

**Important limitations:** the Deco local API is undocumented and may change between firmware versions. The local-API template requires a host-level Secret text password macro and must be validated on the target controller before alert actions are enabled. See the [Deco validation notes](Network_Devices/TP-Link/Deco/TESTING.md).

### SMLIGHT SLZB-MR4U — lightweight HTTP monitoring

The [SMLIGHT SLZB-MR4U template](Network_Devices/SMLIGHT/SLZB-MR4U/README.md) performs **one HTTP GET every two minutes** and uses five dependent items to monitor internal temperature, active socket clients, socket uptime, device uptime and native free-heap values. Four triggers monitor missing metrics, high temperature, zero TCP clients and restarts; three graphs are included. The template was **successfully imported and collected live data on Zabbix 7.0.32**. Set the device-specific `{$SLZB.URL}` macro at host level; no SNMP or agent is required.

**Important limitations:** no individual radio metrics are exposed by the five verified readings; free-heap units remain unconfirmed, and alert transitions have not yet been independently validated. See the [SLZB-MR4U guide](Network_Devices/SMLIGHT/SLZB-MR4U/README.md) for installation, security and caveats.

### TrueNAS SCALE — ZFS ARC-aware memory monitoring

The [TrueNAS SCALE template](Storage_Devices/TrueNAS/template_truenas_scale_by_snmp/README.md) is a modified derivative of the [tr1plus SNMP project](https://github.com/tr1plus/zabbix_truenas_SCALE_snmp). It adds memory metrics that account for ZFS ARC and revised alerts/graphs, while keeping the original TrueNAS SNMP monitoring. Adjusted readings are estimates and **require live validation**.

**License exception:** this directory follows the original [GNU GPL version 2](Storage_Devices/TrueNAS/template_truenas_scale_by_snmp/LICENSE), **not** the root MIT license. See [source attribution and changes](Storage_Devices/TrueNAS/template_truenas_scale_by_snmp/CHANGELOG.md).

## Install a Zabbix template

1. Choose a device from the [template catalog](#available-zabbix-templates).
2. Read its model-specific documentation and download the YAML file.
3. In Zabbix, go to **Data collection → Templates → Import**.
4. Import the template and create a new monitored host.
5. Configure the interfaces and host macros required by the chosen template. For SNMP devices, use appropriate read-only credentials. For SMLIGHT, configure the host-level `{$SLZB.URL}` macro; no agent or SNMP is required.
6. Link the imported template to the host. Inspect **Monitoring → Latest data**, discovery results and unsupported items before configuring notifications.

For the Brother model, follow the [installation and troubleshooting guide](Printers/Brother/DCP-9020CDW/README.md#installation). For TP-Link Deco, follow the [controller and satellite installation guide](Network_Devices/TP-Link/Deco/README.md#installation). For SMLIGHT, follow the [SLZB-MR4U HTTP installation guide](Network_Devices/SMLIGHT/SLZB-MR4U/README.md#installation).

## Compatibility and project status

| Device | Model | Monitoring technology | Status |
| --- | --- | --- | --- |
| Brother | DCP-9020CDW | SNMPv2c / Printer-MIB / IF-MIB / ICMP | Community-developed; collected OIDs verified, Zabbix live import pending |
| TP-Link | Deco mesh systems | ICMP / HTTPS / local encrypted API | Community-developed; controller collection validated on Zabbix 7.0.32; see model/firmware and trigger limitations in the package guide |
| SMLIGHT | SLZB-MR4U | HTTP /metrics | Zabbix 7.0.32 live import and metric collection confirmed; triggers need failure-condition testing |
| TrueNAS | SCALE (upstream target 25.10+) | SNMP / ZFS ARC | GPL v2 derivative; static validation passed, modified template live import pending |

This repository is intended to grow with additional Zabbix templates, organized by **category → manufacturer → model → Zabbix version**. Contributions are welcome, but untested templates will be clearly identified.

## Documentation and contributions

- [Brother DCP-9020CDW: monitoring, installation and FAQ](Printers/Brother/DCP-9020CDW/README.md)
- [Brother DCP-9020CDW: verified SNMP OID reference](Printers/Brother/DCP-9020CDW/OIDS.md)
- [TP-Link Deco: controller and satellite monitoring guide](Network_Devices/TP-Link/Deco/README.md)
- [TP-Link Deco: validation notes](Network_Devices/TP-Link/Deco/TESTING.md)
- [SMLIGHT SLZB-MR4U: HTTP monitoring guide](Network_Devices/SMLIGHT/SLZB-MR4U/README.md)
- [TrueNAS SCALE: monitoring guide](Storage_Devices/TrueNAS/template_truenas_scale_by_snmp/README.md)
- [TrueNAS SCALE: changelog](Storage_Devices/TrueNAS/template_truenas_scale_by_snmp/CHANGELOG.md)
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

**Does this repository provide a SMLIGHT SLZB-MR4U template for Zabbix 7.0?**  
Yes. The [HTTP /metrics YAML](Network_Devices/SMLIGHT/SLZB-MR4U/7.0/template_smlight_slzb_mr4u_by_http.yaml) monitors five verified values using one request every two minutes and Zabbix dependent items. Configure `{$SLZB.URL}` on the host; no SNMP or agent is needed.

**Does this repository provide TP-Link Deco templates for Zabbix 7.0?**  
Yes. The [TP-Link Deco package](Network_Devices/TP-Link/Deco/README.md) contains an agentless ICMP/HTTPS template for every Deco and a separate local-API template for the mesh controller only. Review the documented API and firmware limitations before enabling alert actions.

---

Useful resources: [Zabbix documentation](https://www.zabbix.com/documentation/7.0/en) · [Zabbix community templates](https://github.com/zabbix/community-templates) · [RFC 3805 Printer-MIB](https://www.rfc-editor.org/rfc/rfc3805).
