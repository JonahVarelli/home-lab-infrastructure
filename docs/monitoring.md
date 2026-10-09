# Monitoring and Alerting

## Implementation

Prometheus collects exporter metrics and custom telemetry; Grafana provides dashboards and managed alert rules. The main dashboard separates infrastructure, compute, network, storage, DNS, backup, and monitoring views. A second dashboard was tested at **1280 × 720** for a planned rack display; the physical screen installation remains pending.

## Metrics sources and freshness

| Source | Collection approach | Important interpretation |
| --- | --- | --- |
| Proxmox | API/exporter metrics | Node state is distinct from exporter scrape state |
| AdGuard | Shared exporter for two DNS services | Loss of exporter data can resemble loss of both DNS services |
| TrueNAS | Custom metrics collector surfaced via Node Exporter | `up` for the exporter does not mean TrueNAS data is recent |
| Proxmox backups | Custom collector surfaced via Node Exporter | Last successful backup can remain old after jobs stop running |

TrueNAS telemetry was confirmed in Prometheus under the existing Node Exporter scrape target rather than under a separate `truenas` job. This is why I added separate collector-age alert rules: an exporter may remain responsive while its underlying information stops updating.

## Production alert inventory

All **14 Grafana-managed rules** were listed as **Normal** in the October 9, 2026 review. The table represents the configured monitoring design, not a claim that real outages have been simulated for every rule.

| Rule | Detection condition | Pending period | Severity |
| --- | --- | --- | --- |
| Proxmox node A offline | Health signal below 1 | 5 min | Critical |
| Proxmox node B offline | Health signal below 1 | 5 min | Critical |
| DNS resolver A unavailable | Health signal below 1 | 3 min | Warning |
| DNS resolver B unavailable | Health signal below 1 | 3 min | Warning |
| Both DNS health signals missing | Healthy count below 1 | 2 min | Critical |
| AdGuard exporter offline | Scrape health below 1 | 3 min | Warning |
| Proxmox backup failure | Last job outcome below 1 | 5 min | Critical |
| Backup stale | Oldest reported success older than 36 h | 5 min | Critical |
| Backup coverage incomplete | Fewer than five expected guest records | 5 min | Warning |
| Backup collector stale | Collector age above 10 min | 2 min | Warning |
| ZFS pool unhealthy | Pool health below 1 | 1 min | Critical |
| ZFS capacity warning | Allocation above 80% | 5 min | Warning |
| ZFS capacity critical | Allocation above 90% | 5 min | Critical |
| TrueNAS collector stale | Collector age above 15 min | 2 min | Warning |

## End-to-end notification testing

I configured a combined contact point for Discord and Yahoo email. A temporary Grafana test rule was exercised through the **Firing → Normal** lifecycle, and both channels received firing and recovery notifications. The temporary rule was removed after validation.

That test confirms alert evaluation and delivery for the test condition. It does not establish that every production failure mode triggers as intended, nor can the monitoring container alert externally if it is itself unavailable.

## PromQL: preventing false fallback series

An initial capacity alert added `or vector(999)` directly to a labeled expression. The query unexpectedly returned **two series**: the actual pool allocation near 12.9% and an unlabeled `999` fallback. Grafana correctly classified the `999` as over the threshold; the expression, not the storage, was wrong.

Corrected pattern (public-safe; matches the structure used in the working rule):

```promql
max(
  100 * homelab_truenas_pool_allocated_bytes{pool="tank"}
  / homelab_truenas_pool_size_bytes{pool="tank"}
) or vector(999)
```

Aggregating first yields a single unlabeled value when real data exists and falls back only if the expression has no result. I validated the corrected alert preview showed one series and a **Normal** condition before saving it.

Additional examples are in [PromQL examples](../examples/alert-queries.promql). They illustrate query structure rather than supplying a complete drop-in Grafana configuration.

## Monitoring gaps

- Both ZFS capacity warning and critical alerts may fire above 90%; advanced inhibition and de-duplication have not been independently validated.
- Some missing metric conditions are treated as unhealthy; the matching collector alert is needed to distinguish an outage from stale data.
- No external heartbeat currently verifies the monitoring host from outside the lab.
- Backup collection does not replace an actual restore drill.
- Grafana rule export and screenshot evidence should be sanitized before being added to this public repository.
