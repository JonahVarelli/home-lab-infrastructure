# Network Segmentation and Firewall Policy

## Design goals

The network is divided into zones according to device function and trust level. OPNsense provides VLAN interfaces and policy enforcement; the managed switch and UniFi access point carry the corresponding traffic.

| Zone | Intended role | Access principle |
| --- | --- | --- |
| Trusted | Personal and administrative devices | Internet and explicitly approved internal access |
| Servers | Virtualized services and infrastructure applications | Limited inbound access; controlled outbound connectivity |
| IoT | Smart devices and appliances | Internet and approved services, not unrestricted LAN access |
| Guest | Visitor connectivity | Internet access without administrative or server access |
| Lab | Temporary test workloads | Isolated from normal household and management networks |
| Work | Employer-managed devices | Separate from personal lab and management resources |
| Management | Network and hypervisor administration | Restricted administrative access |

## Policy model

The baseline is **deny inter-VLAN traffic unless required**. Administrative exceptions use defined source and destination aliases; DNS access is handled separately to avoid breaking legitimate name resolution.

Illustrative policy matrix (not a firewall import):

| Source | Destination | Policy intent |
| --- | --- | --- |
| Guest / IoT | Management | Deny |
| Lab | Trusted / Servers / Management | Deny by default |
| Work | Personal lab segments | Deny |
| Approved admin client | Management services | Allow only required services |
| Approved clients | DNS resolvers | Allow DNS according to resolver policy |
| Trusted | Approved server applications | Allow explicitly defined access |

The firewall rule order matters: a broad pass rule above a restrictive rule can defeat the intended policy. When troubleshooting an access issue, I check both the matching rule and the firewall log rather than assuming the destination service is down.

## Switching and wireless

- The firewall-facing switch connection carries tagged VLAN traffic.
- Endpoint-facing access ports use the intended untagged/PVID assignment.
- The UniFi access point uses a dedicated management network and maps wireless SSIDs to their respective client VLANs.
- Link negotiation is verified at the switch and endpoint; a link at 100 Mb/s on a 2.5 GbE port warrants physical-layer checks before changing VLAN configuration.

## Validation approach

| Test | Method | Expected observation |
| --- | --- | --- |
| Wireless VLAN assignment | Join each configured SSID and inspect addressing | Client receives an address from its intended zone |
| Network isolation | Attempt connection from isolated clients to administrative services | Denied unless an explicit exception exists |
| Authorized management | Connect from approved administrative device | Required management service is reachable |
| Firewall rule matching | Inspect live-filter log while repeating test | Log identifies relevant pass/block rule |
| Resolver access | Query approved DNS path | Expected internal and public records resolve |

Addressing, firewall exceptions, and per-port configuration are intentionally omitted from the public repository. They are retained in the private change-controlled operations notes.

## Lessons from implementation

Two practical issues stood out: validating the correct PVID/tag configuration on physical ports, and accounting for DNS when tightening inter-VLAN policies. In both cases I used client-side tests and firewall observations to validate the actual traffic path instead of relying on what the configuration screen appeared to show.
