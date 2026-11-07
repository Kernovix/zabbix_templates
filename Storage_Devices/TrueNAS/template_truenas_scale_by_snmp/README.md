# TrueNAS SCALE by SNMP — ZFS ARC-aware memory monitoring

A **Zabbix 7.0** SNMP template for **TrueNAS SCALE**, based on the original [tr1plus/zabbix_truenas_SCALE_snmp](https://github.com/tr1plus/zabbix_truenas_SCALE_snmp) template and adapted by **Kernovix** to report ZFS ARC-adjusted memory utilization.

> Independent derivative template, not an official Zabbix or TrueNAS integration. **License: GNU GPL version 2**, inherited from upstream (see [LICENSE](LICENSE)). The repository-root MIT license does **not** override it.

## Files

| Path | Description |
| --- | --- |
| [7.0/template_truenas_scale_by_snmp.yaml](7.0/template_truenas_scale_by_snmp.yaml) | Zabbix 7.0 YAML import |
| [CHANGELOG.md](CHANGELOG.md) | Modifications and testing status |
| [LICENSE](LICENSE) | Original upstream GNU GPL v2 license |

## Compatibility

- **Zabbix:** 7.0 template export format. Validate with your server's exact 7.0.x release.
- **TrueNAS:** SCALE 25.10+ (upstream's newer TrueNAS enterprise SNMP MIB layout); not all firmware versions independently validated.
- **Access:** read-only SNMP on UDP/161 and ICMP; SNMP traps (UDP/162) are optional.
- **No agent or ZFS configuration changes** are needed.

This is the full upstream monitoring template, preserving existing SNMP items and discovery for disks, temperatures, interfaces, ZFS pools, datasets, volumes, ARC/L2ARC and ZIL, along with its dashboard. The additional logic only uses items already collected over SNMP.

## Memory item reference

| Key | Units | Description |
| --- | --- | --- |
| `vm.memory.total` | B | Total physical memory (upstream item) |
| `vm.memory.available` | B | Upstream SNMP approximation: free + buffers + cached |
| `vm.memory.util` | % | Unmodified *raw* utilization calculation; kept for comparison |
| `truenas.zfs.arc.size` | B | Current ZFS ARC size (upstream item) |
| `vm.memory.available.zfs` | B | **Added:** estimated available memory including reclaimable ARC |
| `vm.memory.used.system.zfs` | B | **Added:** estimated non-ARC system RAM consumption |
| `vm.memory.util.zfs` | % | **Added:** estimated RAM utilization excluding ARC |

### Calculated-item expressions

Adjusted available memory (capped to physical memory):

```text
min(last(//vm.memory.total),
    last(//vm.memory.available) + last(//truenas.zfs.arc.size))
```

Estimated memory used by the system outside ARC:

```text
last(//vm.memory.total) - last(//vm.memory.available.zfs)
```

Estimated percent utilization excluding ARC:

```text
100 * (last(//vm.memory.total) - last(//vm.memory.available.zfs))
    / last(//vm.memory.total)
```

Zabbix 7.0 supports the mathematical `min(value1,value2)` function in calculated items.

## Updated triggers

| Condition | Default behavior |
| --- | --- |
| High memory usage | Adjusted `vm.memory.util.zfs` **above 90% continuously for 5 minutes** |
| Recovery | Adjusted utilization **below 85% continuously for 5 minutes** |
| Critically low memory | Adjusted available RAM **below 512 MiB for 5 minutes** (High) |
| Swap occupancy | Upstream threshold of **less than 50% swap free** for 5 minutes; now independent of the RAM alarms |

The original raw memory item remains available; the memory graphs compare raw/adjusted values, non-ARC system memory and ZFS ARC. Other discovery rules and monitoring components are preserved.

### Important limitations

**These values are estimates—not direct Linux kernel MemAvailable, actual process RSS or a measure of memory pressure.** The ARC-adjusted calculation assumes ARC is largely reclaimable **and** that the SNMP `available` value does not already include the ARC. If any ARC is already reflected in SNMP cached/available memory, the calculation can **double-count** it, overestimate available RAM and hide real memory pressure. Some ARC cannot shrink immediately. The system-memory item is not a precise process-usage counter.

Do not interpret the adjusted percentage alone as proof of health. Check raw memory, ARC, swap, TrueNAS graphs and application performance. The template does **not** directly measure swap-in/out or Linux PSI. The 512 MiB threshold is a starting point for a system with approximately 16 GiB RAM, not a universal safety threshold. Confirm that ARC size units correspond to the TrueNAS UI.

## Import or update

1. Back up your current Zabbix template (especially if customized).
2. Enable SNMP in TrueNAS, and ensure Zabbix can reach its read-only SNMP interface.
3. Use **Data collection → Templates → Import** and select [the YAML](7.0/template_truenas_scale_by_snmp.yaml).
4. Enable **Create new** / **Update existing**, keep **Delete missing** off. The original template name `TrueNAS SCALE by SNMP` and existing UUIDs are preserved for in-place upgrades.
5. Link the template, check **Monitoring → Latest data**, confirm raw SNMP memory and ARC values, then validate `vm.memory.available.zfs`, `vm.memory.used.system.zfs` and `vm.memory.util.zfs`.
6. Review the `{$MEMORY.UTIL.MAX}`, `{$MEMORY.UTIL.RECOVERY}`, `{$MEMORY.AVAILABLE.MIN}` and `{$SWAP.PFREE.MIN.WARN}` macros per host before relying on alert notifications.

## Validation status

- YAML syntax and structure, duplicate UUIDs, local item references and graph references: **validated**.
- Real import of the **modified** YAML into a running Zabbix 7.0 instance and end-to-end alert/recovery behavior: **not yet verified**.
- Original project's compatibility tests are not proof of validation of these changes.

## Source, changes and license

- **Original source / author:** [tr1plus/zabbix_truenas_SCALE_snmp](https://github.com/tr1plus/zabbix_truenas_SCALE_snmp).
- **Upstream source file:** [template_app_truenas_scale_snmp.yaml](https://github.com/tr1plus/zabbix_truenas_SCALE_snmp/blob/main/template_app_truenas_scale_snmp.yaml).
- **Base source blob SHA:** `a52070992948b6586b34d01bdf537900319146b8`.
- **Directory structure reference:** [zabbix/community-templates — TrueNAS / 7.0](https://github.com/zabbix/community-templates/tree/main/Storage_Devices/TrueNAS/template_truenas_scale_by_snmp/7.0).
- **Kernovix changes:** 2026-11-07; detailed in [CHANGELOG.md](CHANGELOG.md).
- **License:** GNU General Public License, version 2, copied verbatim from upstream into [LICENSE](LICENSE). This derivative is **not** MIT-licensed merely because the repository's main LICENSE is MIT.

Never post SNMP community strings, passwords, private addresses or full unsanitized SNMP walks in public issues.
