# Brother DCP-9020CDW: SNMP OID inventory

These OIDs were observed on an NC-8100w firmware 1.16 Brother DCP-9020CDW. Values can differ across firmware versions.

| Metric | OID | Notes |
| --- | --- | --- |
| System description | `1.3.6.1.2.1.1.1.0` | Firmware |
| System uptime | `1.3.6.1.2.1.1.3.0` | Timeticks; divide by 100 for seconds |
| Printer model | `1.3.6.1.2.1.25.3.2.1.3.1` | Device description |
| Printer status | `1.3.6.1.2.1.25.3.5.1.1.1` | Host Resources MIB |
| Printer error bitmap | `1.3.6.1.2.1.25.3.5.1.2.1` | OCTET STRING |
| Total page count | `1.3.6.1.2.1.43.10.2.1.4.1.1` | Printer-MIB |
| Tray maximum | `1.3.6.1.2.1.43.8.2.1.9.1.1` | 250 observed |
| Tray remaining | `1.3.6.1.2.1.43.8.2.1.10.1.1` | -3 = nonempty, unknown amount |
| Alert code | `1.3.6.1.2.1.43.18.1.1.7.1.1` | Event-dependent |
| Supply description | `1.3.6.1.2.1.43.11.1.1.6.1.N` | N=1..10 |
| Supply maximum | `1.3.6.1.2.1.43.11.1.1.8.1.N` | N=6 belt, N=7..10 drums |
| Supply current | `1.3.6.1.2.1.43.11.1.1.9.1.N` | N=6 belt, N=7..10 drums |
| Private maintenance TLV | `1.3.6.1.4.1.2435.2.3.9.4.2.1.5.5.8.0` | CMYK toner tags 0x81..0x84 |
| Interface description | `1.3.6.1.2.1.2.2.1.2` | Use IF-MIB discovery |
| Interface operational status | `1.3.6.1.2.1.2.2.1.8.N` | 1 up, 2 down |
| Interface receive octets | `1.3.6.1.2.1.2.2.1.10.N` | Counter32 |
| Interface transmit octets | `1.3.6.1.2.1.2.2.1.16.N` | Counter32 |

Caution: toner readings in the generic Printer-MIB are negative status codes, not percentages. Private maintenance values need Zabbix preprocessing validation.
