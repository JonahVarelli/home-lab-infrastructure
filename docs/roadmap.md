# Roadmap and Technical Debt

## Current baseline

At the October 2026 checkpoint, network segmentation, Proxmox, TrueNAS, dual AdGuard resolvers, scheduled backups, and centralized monitoring are operating. The main and compact Grafana dashboards are built; fourteen production alerts were Normal during the recorded audit.

## Planned work

| Priority | Change | Definition of done |
| --- | --- | --- |
| 1 | ISP gateway bridge-mode change | Planned cutover completed with rollback path and validated WAN, DNS, firewall and VLAN behavior |
| 2 | WireGuard remote access | External access works only for authorized clients and intended destinations |
| 3 | Backup restore drill | Isolated restore boots and passes data/application checks without disturbing the source guest |
| 4 | Physical rack display | Install screen, configure kiosk startup and verify 1280 × 720 visibility and reboot recovery |
| 5 | Photo management / service growth | Deploy only after resource, storage, access and backup requirements are defined |
| 6 | Independent monitoring heartbeat | Detect loss of the Grafana/Prometheus host or outbound alert delivery |

## Known limitations

- **Two-node cluster:** Requires deliberate quorum and recovery planning; not a claim of automated HA.
- **Storage:** The two-disk ZFS mirror is local redundancy, not a complete backup strategy.
- **Backups:** Archive creation is monitored, but end-to-end recovery remains unproven until the restore test.
- **Monitoring:** A local monitoring failure can prevent alert delivery. Combined failure modes can also cause overlapping notifications.
- **DNS:** Multiple resolvers reduce single-instance dependence but do not guarantee how every client will select a resolver.

## Change documentation standard

For significant changes I track: the initial condition, intended change, affected services, implementation notes, a rollback plan, the validation method, and the observed result. Only sanitized summaries are published here; operational details remain in private documentation.
