# SMLIGHT SLZB-MR4U — Zabbix 7.0 HTTP Monitoring

**[Download Zabbix 7.0 YAML template](7.0/template_smlight_slzb_mr4u_by_http.yaml)**

A lightweight, **agentless** monitoring template for the SMLIGHT SLZB-MR4U network Zigbee/Thread adapter. It reads the firmware's built-in `/metrics` endpoint over HTTP. This package was validated on **Zabbix 7.0.32** with a live SLZB-MR4U on **2026-11-12**: template import and all five dependent items were confirmed in Latest data.

> **Independent community template.** Not affiliated with or endorsed by SMLIGHT or Zabbix SIA. Actual alert/failure transitions and compatibility across firmware releases have not yet been tested.

## Requirements

- Zabbix **7.0** (tested with 7.0.32).
- SMLIGHT SLZB-MR4U running SLZB-OS with an accessible HTTP `/metrics` endpoint.
- Network access **from the Zabbix server or proxy** to the coordinator's management HTTP port (typically TCP/80).
- No Zabbix Agent, SNMP, external Prometheus server, Python, or shell scripts required.

## Installation

1. Import [template_smlight_slzb_mr4u_by_http.yaml](7.0/template_smlight_slzb_mr4u_by_http.yaml) from **Data collection → Templates → Import**.
2. Create a host (for example, `SMLIGHT SLZB-MR4U`) and link **SMLIGHT SLZB-MR4U by HTTP**. A host agent/SNMP interface is **not required**.
3. On the host, override the user macro **`{$SLZB.URL}`** with its real management address, for example `http://YOUR_SLZB_IP/metrics`. The template's placeholder is deliberately not a real IP address.
4. Confirm **Monitoring → Latest data**. The single HTTP master item should return the Prometheus-style text; the five dependent items should have numeric values.
5. Review **Data collection → Hosts → Triggers** and **Monitoring → Hosts → Graphs** before configuring notifications.

Quick diagnostic from the Zabbix server or proxy:

```bash
curl -fsS --max-time 5 http://YOUR_SLZB_IP/metrics
```

## Low-overhead design

- **One HTTP GET every 120 seconds** (30 requests/hour per monitored host).
- **Five dependent items**, extracted locally by Zabbix from a single source response; no additional device requests.
- **No active network scanning, discovery, socket probes, or automated device actions.**
- Short raw-text retention (**1 day**), with 30-day item history and 365-day trends for numeric items.

## Items

| Item key | Metric name | Notes |
| --- | --- | --- |
| `slzb.metrics.raw` | Entire `/metrics` response | HTTP master item; collected every 2 minutes |
| `slzb.socket.clients` | `smlight_socket_clients` | TCP socket clients; **not** Zigbee paired-device count |
| `slzb.socket.uptime` | `smlight_socket_uptime` | Seconds |
| `slzb.device.uptime` | `smlight_device_uptime` | Milliseconds converted to seconds |
| `slzb.device.temperature` | `smlight_device_temp` | Celsius |
| `slzb.device.heap_free` | `smlight_free_heap` | Original firmware unit; not confirmed and therefore not converted |

The five numeric metrics above were observed on the tested device. Firmware builds may expose different or additional metrics. No individual-radio uptime, radio temperature, Thread state, or connected Zigbee-device count is inferred from these data.

## Trigger defaults

| Trigger | Condition | Severity |
| --- | --- | --- |
| No metrics | No device-temperature value for **12 minutes** | High |
| Sustained high temperature | Minimum temperature over **10 minutes** is above `{$SLZB.TEMP.WARN}` | Warning |
| No TCP clients | Maximum client count over **10 minutes** is zero | Warning |
| Device restart | Device uptime decreased between samples | Information |

The default warning threshold **`{$SLZB.TEMP.WARN}=70` °C** is a configurable operational threshold, **not an official SMLIGHT hardware limit**. A zero TCP client count can be normal if no client is expected to be connected; disable or adjust that trigger in such deployments. All trigger transitions and recovery conditions should be tested locally before integrating with paging.

## Graphs

- **SLZB: internal temperature**
- **SLZB: active TCP clients**
- **SLZB: free heap (native value)**

## Security and privacy

The firmware's `/metrics` endpoint is polled over plain HTTP in this template. Restrict management access to a trusted network and enable web UI authentication when supported and appropriate. Avoid exposing the endpoint publicly. The published template does **not** contain real management IP addresses, secrets, device IDs, or private network inventory.

## Known limitations

- Only the **five verified** metrics are implemented; there is no claim of comprehensive radio-level monitoring.
- The meaning/unit of `smlight_free_heap` needs vendor confirmation; no memory alert is included.
- No SNMP, ICMP, TCP-port monitoring, or automatic reboots are part of this template.
- Zabbix 7.0.32 successfully imported and collected all six items (one master + five dependent). Threshold activation, trigger recovery, long-term trends and behavior on other firmware versions are **not yet independently validated**.

## References

- [SMLIGHT manuals](https://smlight.tech/support/manuals/)
- [Zabbix 7.0 HTTP agent documentation](https://www.zabbix.com/documentation/7.0/en/manual/config/items/itemtypes/http)
- [Zabbix 7.0 dependent items](https://www.zabbix.com/documentation/7.0/en/manual/config/items/itemtypes/dependent_items)

Community-maintained under the repository [MIT license](../../../../LICENSE). Please report issues with firmware and Zabbix versions, item keys, and sanitized diagnostics; never post private IP addresses or full configuration backups.
