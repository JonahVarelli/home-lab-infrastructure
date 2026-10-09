# Implementation Validation

This document separates **observed results** from **pending tests**. Dates and results reflect the October 2026 documentation checkpoint, not continuous guarantees of uptime.

## Verified at the checkpoint

| Area | Verification method | Observation |
| --- | --- | --- |
| VLAN and wireless separation | Client addressing, permitted/blocked connectivity, firewall logs | Clients joined intended zones and selected isolation paths behaved as designed |
| Proxmox | Host reachability, cluster interface, node metrics | Both nodes operating; final alert inventory showed both healthy |
| TrueNAS | ZFS API/collector metric checks | Mirror health metric healthy and pool utilization approximately 12.9% during alert setup |
| NFS backup storage | Authorized-host connectivity and write checks | TrueNAS export accessible from Proxmox clients |
| Scheduled backups | Proxmox job output, archive and telemetry checks | Five expected production-guest backup records present; last recorded backup age within the configured freshness window |
| DNS | Resolver queries, client behavior, exporter metrics | Two AdGuard instances reporting healthy at the monitored checkpoint |
| Alert evaluation | Grafana Alert rules inventory | Fourteen production rules in Normal state |
| Notification pipeline | Temporary Grafana rule fired and resolved | Discord and email both received firing and recovery messages |
| Rack Status display design | Browser viewport measured at 1280 × 720 | Dashboard layout visually tested at target resolution |

## Outstanding validation

| Test | Status | Acceptance criterion |
| --- | --- | --- |
| Isolated Proxmox restore | Not performed | Restored guest boots, data is present, intended services respond, original guest unaffected |
| ISP bridge-mode cutover | Planned | OPNsense obtains expected WAN connectivity; VLAN/DNS/security regression tests pass |
| WireGuard remote access | Planned | Authenticated external connection reaches only approved destinations |
| Physical rack display | Pending hardware integration | Dashboard remains legible after installation, reboot and kiosk recovery |
| Monitoring host failure detection | Not configured externally | Independent system reports loss of monitoring host or its notification path |
| Alert dependencies | Not fully validated | Appropriate suppression/grouping behavior across overlapping fault conditions |

## Repeatable checks

Example read-only client checks (run from the relevant client and compare against its assigned zone):

```powershell
ipconfig /all
nslookup example.com
```

Example status checks from an authorized administrator session:

```bash
# Run on a Proxmox host
pvecm status

# Run in an authorized TrueNAS shell
zpool status tank
```

These commands demonstrate inspection methods. They do not provide private access paths or replace system-specific authorization and change-control procedures.

## Evidence handling

Before adding screenshots or outputs to this repository, redact management URLs, private host addresses, account identifiers, tokens, keys, webhook addresses, and other information unrelated to demonstrating the test. A successful check should include what was tested, the expected behavior, and the observed result—not only a green dashboard indicator.
