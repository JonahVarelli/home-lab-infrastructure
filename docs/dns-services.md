# DNS and Application Services

## DNS design

I deployed two AdGuard Home instances as LXCs on different Proxmox nodes. They provide DNS filtering for clients on the segmented network. Internal names use the designated local resolution path, while external queries use configured upstream resolvers.

| Concern | Implementation | Constraint |
| --- | --- | --- |
| Resolver availability | Two AdGuard instances | Clients may not fail over in a predictable order |
| Internal names | Local DNS integration | Depends on correct forward/reverse resolution configuration |
| Access controls | Firewall and resolver policies | Rules must allow required DNS from each permitted zone |
| Filtering | AdGuard Home | Encrypted application-level DNS can bypass conventional port 53 enforcement |
| Monitoring | Exporter data from both resolvers | Both health signals rely on a shared metrics exporter |

Resolver availability was checked from the client side, not just by looking at container state. The monitoring configuration includes separate per-instance rules, a combined health rule, and an exporter-health rule so a collector failure is distinguishable from a confirmed service outage.

## Application placement

Application services run primarily inside a Debian VM using Docker. The environment includes a media service stack and supporting applications. Service placement keeps application dependencies separate from the Proxmox host operating systems.

A running container is not sufficient proof that a service is usable. Checks include DNS resolution, network policy, storage mounts, and file permissions. I avoid treating a successful container startup as equivalent to a verified end-user workflow.

## Security and operations

- Administrative interfaces are not generally exposed to guest or IoT networks.
- The DNS service is available to approved zones without granting those zones broad access to infrastructure management.
- Application credentials, internal records, runtime environment files, and live service configuration are not published here.
- DNS redundancy improves resilience, but it is not a guarantee of instantaneous client failover or complete DNS bypass prevention.
