# Changelog — TrueNAS SCALE SNMP template (Kernovix derivative)

## 2026-11-07 — ZFS ARC-aware memory monitoring

Base: original [tr1plus/zabbix_truenas_SCALE_snmp](https://github.com/tr1plus/zabbix_truenas_SCALE_snmp) `template_app_truenas_scale_snmp.yaml`; upstream Git blob `a52070992948b6586b34d01bdf537900319146b8`.

### Additions

- `vm.memory.available.zfs`: estimated available RAM = `min(total, raw available + ARC size)`.
- `vm.memory.used.system.zfs`: estimated non-ARC system RAM use, in bytes.
- `vm.memory.util.zfs`: estimated utilization excluding the ARC, in percent.
- Graph comparing system memory estimate and ZFS ARC.

### Adjustments

- Kept the raw `vm.memory.util` metric for comparison but changed the high-memory trigger to use `vm.memory.util.zfs` (default >90% for 5m).
- Added explicit recovery: adjusted memory utilization <85% for 5m.
- Changed the low-available-memory alert to the new adjusted item, with 512 MiB default threshold and HIGH severity (was 20 MiB).
- Made the upstream swap-occupancy alert independent of the high-memory/low-memory triggers.
- Extended memory graphs to show original and adjusted measurements.
- Preserved upstream OIDs, original template name, original item UUIDs and all unrelated monitoring functionality.

### Known limitations / verification

- SNMP cached-memory values may already include some ARC; the ARC-adjusted estimate may **double count** available memory. It is not a replacement for real memory pressure monitoring.
- Static checks passed (YAML parse, UUID uniqueness, referenced calculated items and graph items).
- Zabbix 7.0 live import, readings and recovery behavior of this modified template remain **unverified**.

**Original author:** tr1plus. **Modifications:** Kernovix. **License:** upstream GNU GPL version 2; see [LICENSE](LICENSE).
