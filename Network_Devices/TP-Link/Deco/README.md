# TP-Link Deco Zabbix 7.0 Monitoring Templates

Community Zabbix templates for TP-Link Deco mesh systems. The package deliberately separates **node availability** from **controller-only mesh telemetry** so that every Deco can be monitored without creating unnecessary authenticated sessions.

**[Basic ICMP/HTTPS template](7.0/template_tplink_deco_by_icmp_https.yaml)** · **[Controller local-API template](7.0/template_tplink_deco_by_local_api.yaml)** · **[Validation notes](TESTING.md)**

> **Status: community-developed.** The authenticated local API flow has been collected successfully from a live Deco controller on Zabbix 7.0.32. The TP-Link local API is undocumented and may differ by model or firmware. Validate items, discovery and trigger behaviour in a non-production alerting window before relying on notifications.

This is an independent community project. It is not affiliated with, sponsored by, or endorsed by TP-Link or Zabbix SIA.

## Choose the right template

| Template | Link it to | Credentials | What it monitors |
| --- | --- | --- | --- |
| `TP-Link Deco by ICMP and HTTPS` | **Every Deco**, including the controller and all satellites | None | ICMP reachability, packet loss, response time and HTTPS management-page availability |
| `TP-Link Deco by local API` | **Only the primary/controller Deco** | Deco owner password in a secret host macro | Controller CPU/memory, online clients, mesh inventory, per-node state, model, firmware, role, IP address, connection type, radio signal level and backhaul link speed |

Use the basic template on every mesh node. The local-API template must not be linked to individual satellite Decos: they do not reliably serve mesh-wide inventory/performance endpoints and each linked host would create another authenticated session.

## Requirements

- **Zabbix:** 7.0 or later.
- **Connectivity:** the Zabbix server or selected proxy must reach every monitored Deco over ICMP and HTTPS.
- **Host interface:** create a host interface that resolves to the Deco management IP/DNS name. No Zabbix agent is installed or required.
- **Controller API:** HTTPS local management must be enabled/reachable. The template uses the local web UI protocol; SNMP and CWMP are not required.
- **Certificate:** Deco devices commonly use a self-signed certificate. The basic HTTPS check intentionally does not verify the peer or host name. Use a trusted certificate and change the check if your environment requires certificate verification.

## Installation

### 1. Monitor every Deco

1. In **Data collection → Templates → Import**, import [template_tplink_deco_by_icmp_https.yaml](7.0/template_tplink_deco_by_icmp_https.yaml).
2. Create one Zabbix host for each physical Deco. Use the Deco IP address or a resolvable local DNS name as its host interface.
3. Link **TP-Link Deco by ICMP and HTTPS** to each host.
4. Review **Monitoring → Latest data** after several minutes.

No password, SNMP community, agent, CWMP configuration or external script is needed for this template.

### 2. Add controller telemetry (optional)

1. Identify the mesh controller/primary Deco in the Deco application or local web interface.
2. Import [template_tplink_deco_by_local_api.yaml](7.0/template_tplink_deco_by_local_api.yaml).
3. Link **TP-Link Deco by local API** to the controller host only. It can coexist with the ICMP/HTTPS template on that host.
4. Add the host-level macro below. Select the **Secret text** macro type in the Zabbix UI.

| Macro | Required value | Purpose |
| --- | --- | --- |
| `{$DECO.PASSWORD}` | Deco owner password | Authenticates the local API session |
| `{$DECO.SCHEME}` | `https` (default) | Management-interface scheme |

5. Wait for the collection interval, or use **Execute now** for `Deco API compact source data`.
6. Confirm that `CPU utilization`, `Memory utilization`, `Online clients`, `Mesh nodes online` and the discovered `Deco <node>: ...` items have values.

The API item downloads the encryption code exposed by the Deco local web UI, creates an encrypted session, collects three read-only endpoints, stores a compact normalized result, then sends a best-effort logout for the session it created. The password is not written to the template or raw item history.

## Collected data

### All Decos: basic template

| Area | Item | Polling / retention |
| --- | --- | --- |
| Reachability | ICMP ping | Every 30 seconds / 7 days |
| Quality | ICMP packet loss and response time | Every 30 seconds / 7 days plus 365-day trends |
| Management UI | HTTPS management interface | Every minute / 7 days |

### Controller only: local API template

| Area | Items |
| --- | --- |
| Controller health | CPU utilization and memory utilization |
| Mesh summary | Mesh node count and mesh nodes online |
| Clients | Online client count |
| Auto-discovered per-node inventory | Online state, role, IP address, model and firmware |
| Auto-discovered per-node link data | Connection type, 2.4 GHz signal level, 5 GHz signal level and reported backhaul speed |

`Deco API compact source data` is the dependent-item source. It is intentionally retained for one day only and contains a normalized mesh summary, not client lists, LAN-port MAC-address lists, device IDs or other bulky UI data.

The radio signal values are vendor-reported levels, not calibrated dBm/RSSI measurements. A value of `0` can mean that the field is unavailable, wired, or not meaningful for the primary node. The reported backhaul speed is link data from the Deco API, not an internet speed test. Neither has an automatic problem trigger.

## Triggers and false-positive protection

The templates favour confirmed faults over noisy alerts.

| Template | Problem | Default condition | Severity |
| --- | --- | --- | --- |
| ICMP/HTTPS | Deco unavailable by ICMP | Three consecutive failed 30-second probes | High |
| ICMP/HTTPS | High packet loss | Every value over `{$ICMP.LOSS.WARN}` for 5 minutes | Warning |
| ICMP/HTTPS | High response time | 5-minute average over `{$ICMP.RESPONSE.WARN}` | Warning |
| ICMP/HTTPS | HTTPS management interface unavailable | No accepted HTTPS result for 5 minutes | Average |
| Local API | API has no data | No successful collection for `{$DECO.API.NODATA}` | High |
| Local API | High CPU or memory | Last three successful readings all above the configured threshold | Warning |
| Local API | Mesh node offline | Two consecutive successful controller collections report the node offline | High |

The controller defaults are deliberately conservative: a fixed 15-minute API interval and a one-hour no-data period. A transient Deco API timeout therefore does not turn into a false node-offline, CPU, or memory problem.

## Local API macros

| Macro | Default | Meaning |
| --- | ---: | --- |
| `{$DECO.API.NODATA}` | `1h` | Time without a successful API collection before an API problem opens. |
| `{$DECO.CPU.WARN}` | `90` | CPU percentage required in all of the last three successful samples. |
| `{$DECO.MEMORY.WARN}` | `90` | Memory percentage required in all of the last three successful samples. |
| `{$DECO.PASSWORD}` | *(empty secret)* | Required Deco owner password; configure at host level. |
| `{$DECO.SCHEME}` | `https` | Management interface scheme. |

The local-API item interval is fixed at **15 minutes** in the template. This prevents an unresolved user macro from making the item unsupported and avoids unnecessary Deco sessions. If a different interval is required, change the item delay directly in a lab-tested copy of the template and keep it conservative.

## Basic-template macros

| Macro | Default | Meaning |
| --- | ---: | --- |
| `{$ICMP.LOSS.WARN}` | `20` | Packet-loss warning threshold in percent. |
| `{$ICMP.RESPONSE.WARN}` | `0.2` | ICMP response-time warning threshold in seconds. |

## Graphs

The local-API template includes:

- **TP-Link Deco: CPU and memory utilization**
- **TP-Link Deco: mesh nodes and clients**

Per-node signal and backhaul values remain as individual latest-data/history items because their scales and availability differ between wired, wireless and primary nodes.

## Troubleshooting

**`CryptoJS undefined` or a similar JavaScript error.** Import the current local-API YAML. The crypto libraries must be evaluated in the Script item’s main Duktape scope; this version does so.

**`Deco API ... timeout`.** Keep the local-API template linked only to the controller, confirm that Zabbix can reach its HTTPS UI, and do not shorten `{$DECO.API.INTERVAL}`. The template will not mark mesh nodes offline from a failed collection; it opens the API no-data problem only after the configured delay.

**The API items are empty on a satellite Deco.** This is expected on firmware that serves mesh-wide endpoints only on the controller. Use the ICMP/HTTPS template on satellites.

**HTTPS item is unsupported.** Check the management URL from the Zabbix server/proxy, firewall routing and the host interface. The template accepts `200`, `301`, `302`, `401` and `403` response codes and ignores self-signed certificate validation by design.

**Signal/backhaul data is zero or unavailable.** This is not an availability fault. Firmware, wired backhaul and primary-node topology can omit these values.

## Security and privacy

- Set `{$DECO.PASSWORD}` as a Zabbix **Secret text** macro at host level. Never add it to YAML, documentation, issue reports or screenshots.
- Do not publish raw API records: they may contain MAC addresses, device identifiers, local IP addresses and client inventory information.
- The local API is undocumented. Use it only on a trusted LAN and review updates after a Deco firmware upgrade.

## Compatibility and scope

The authenticated collection was observed against a TP-Link Deco mesh containing M4R and E4R units and Zabbix 7.0.32. The template has not been validated against every Deco family, firmware branch, standalone router mode or cloud-managed deployment. TP-Link can change the local UI/API without notice.

## References

- [Zabbix 7.0 template import/export documentation](https://www.zabbix.com/documentation/7.0/en/manual/xml_export_import/templates)
- [Zabbix script items](https://www.zabbix.com/documentation/7.0/en/manual/config/items/itemtypes/script)
- [Zabbix community templates](https://github.com/zabbix/community-templates)
- [ha-tplink-deco](https://github.com/amosyuen/ha-tplink-deco) — reference implementation for Deco local polling behaviour

Community-maintained under the repository [MIT license](../../../LICENSE). Please report issues with the exact Deco model, firmware, Zabbix version, affected item key and a sanitized error message.
