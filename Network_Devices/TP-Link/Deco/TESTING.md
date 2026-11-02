# TP-Link Deco Template Validation Notes

This document records the validation status of the templates in this directory. It intentionally excludes real host names, management addresses, passwords, MAC addresses and raw API responses.

## Environment observed

| Component | Observed value |
| --- | --- |
| Zabbix frontend/server | 7.0.32 |
| Deco API transport | Local HTTPS UI with encrypted session |
| Mesh topology | One controller plus four satellite Decos |
| Observed Deco models | M4R and E4R |
| Observed firmware family | 1.6.x, 1.7.x and 1.8.x |

## Successful checks

- The local-API template imported into the live Zabbix 7.0.32 instance.
- The Script item authenticated to the controller and returned CPU and memory utilization, mesh-node inventory and online-client count.
- Low-level discovery created per-node model, firmware and online-state items for all five mesh nodes.
- Controller API data reported all expected nodes online and produced the two template graphs.
- ICMP and HTTPS reachability were verified from the monitoring environment for each of the five Deco management addresses.
- YAML parsing and JavaScript syntax checks were run after each template revision.

## Important limitations

- The local API is controller-only in the tested topology. Satellite Decos accepted the authentication handshake but did not reliably return inventory/performance data.
- The Deco firmware can return transient `timeout` API responses. The template treats a failed collection as no data; it does not convert the failure into zero online nodes or zero utilization.
- The updated compact-source layout and additional per-node link items should be imported and verified after any local modification before publication.
- Fault injection was not performed: ICMP loss, an offline node, high utilization and API no-data recovery should be tested in a controlled maintenance window before enabling production actions.

## Publication checklist

Before merging this package into the repository:

1. Import both YAML files into a Zabbix 7.0 test instance.
2. Link the ICMP/HTTPS template to a controller and at least one satellite.
3. Link the local-API template only to the controller, set a Secret text password macro and force a collection.
4. Confirm compact source data and all expected discovery items.
5. Confirm that the two graphs resolve to the new template items.
6. Deliberately test or carefully simulate each trigger/recovery path where operationally safe.
7. Ensure no raw API history, screenshots, passwords, IP addresses, MAC addresses or serial/device IDs are committed.
