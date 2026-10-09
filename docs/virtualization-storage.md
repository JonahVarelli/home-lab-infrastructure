# Virtualization, Storage, and Backups

## Proxmox VE

The compute environment consists of two small-form-factor Proxmox hosts. VMs and Linux containers support network services, monitoring, application hosting, and isolated test workloads.

| Workload class | Placement approach | Rationale |
| --- | --- | --- |
| Infrastructure applications | Dedicated VM/LXC where appropriate | Easier lifecycle management and troubleshooting |
| DNS | Two LXCs on separate Proxmox nodes | Reduce dependency on a single compute node |
| Monitoring | Dedicated LXC | Central place for metrics, dashboards and alert evaluation |
| Docker applications | Debian VM | Separate application stack from Proxmox host OS |

Both nodes participate in the cluster. **This is not advertised as high availability**: two-node quorum, application placement, local storage, and failover behavior require independent consideration. Earlier instability on one node was investigated at the firmware, networking, and thermal layers; the node was later reported operational. The incident record avoids attributing every symptom to a single unverified root cause.

## TrueNAS and ZFS

TrueNAS runs on a separate machine with **two 14 TB data drives in a ZFS mirror**. The mirror can tolerate a single drive failure while the remaining member is healthy, but it does not protect against every failure mode or replace independent backups.

TrueNAS provides SMB file sharing and an NFS export used as the Proxmox backup destination. NFS host authorization and protocol configuration were verified during initial storage integration.

## Backup implementation

| Setting | Implemented configuration |
| --- | --- |
| Schedule | Daily at 03:00 local time |
| Destination | TrueNAS NFS storage |
| Backup mode | Snapshot |
| Compression | ZSTD |
| Protected guests | Five production VMs/LXCs |
| Exclusion | Disposable test workload |
| Retention | Seven daily, four weekly, three monthly restore points |

I check backup job outcomes and archive reporting, and Prometheus collects additional signals: last reported job success, age of successful backups, coverage of the expected guest set, and freshness of the backup metrics collector.

**Validation boundary:** Completed jobs and visible archives demonstrate that the scheduled backup workflow is functioning, but an isolated guest restore has **not** yet been completed. Restore testing is explicitly tracked as outstanding work.

## Operational considerations

- **Storage:** A mirrored pool is a local availability measure, not an off-site or independent backup.
- **Compute:** Mini-PC hardware requires attention to memory capacity, firmware revisions, cooling, and NIC behavior.
- **Recoverability:** A meaningful restore test must verify guest startup, data integrity, and service function, not only archive extraction.
- **Observability:** Collector freshness matters. Old success metrics cannot be treated as evidence that today's backup ran.

See [Monitoring and alerting](monitoring.md), [Validation](validation.md), and [Incident reports](incident-reports.md).
