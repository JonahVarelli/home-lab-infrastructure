# Infrastructure Incident Reports

These reports document specific observations, diagnostic decisions, and validation steps from the build. They are summarized for public review; sensitive configuration and identifying details remain in private operational notes.

## INC-01 — ZFS capacity alert produced a false firing series

**Impact:** Alert preview showed a critical-looking result while ZFS utilization was approximately 13%; no corresponding storage-capacity incident was observed.

**Symptoms:** Prometheus returned both the true utilization and a separate fallback series valued at `999`.

**Investigation:** I inspected the raw query output rather than relying on the alert status alone. The real series carried target/pool labels, whereas `vector(999)` was unlabeled. PromQL's `or` therefore returned both series.

**Resolution:** Aggregated the capacity calculation with `max(...)` before applying `or vector(999)`.

**Validation:** The revised query returned one series near actual utilization. The preview condition was false and displayed **Normal** before the rule was saved.

**Lesson:** Verify cardinality and labels in addition to numeric values when developing alert expressions.

## INC-02 — Managed switch port negotiated at 100 Mb/s

**Impact:** An endpoint connected through a 2.5 GbE switch port negotiated at 100 Mb/s, reducing available bandwidth.

**Symptoms:** Multiple devices showed the same low negotiated speed on one port; other ports were unaffected.

**Investigation:** Compared device, cable, and port combinations to separate configuration problems from a physical fault. The issue was traced to damage associated with a misconnected PoE injector lead.

**Resolution:** Moved the access point to a known-good port and replaced affected hardware as appropriate.

**Validation:** The service path was restored through a working port; the damaged port was removed from use.

**Lesson:** Check Layer 1 first when a speed or link issue follows a specific physical interface.

## INC-03 — Proxmox could not use the TrueNAS NFS backup export

**Impact:** Backup storage was not available to the Proxmox nodes in the intended configuration.

**Symptoms:** Proxmox storage integration failed despite the NAS being accessible for administration.

**Investigation:** Checked export permissions, protocol settings, NFS service availability, and reachability from each authorized client.

**Resolution:** Corrected export configuration, including NFSv4.2 and authorized Proxmox hosts.

**Validation:** Verified NFS access and writes from the designated hosts, then confirmed backup storage integration.

**Lesson:** A reachable management interface does not prove the service protocol and client permissions are correct.

## INC-04 — Intermittent Proxmox node instability

**Impact:** One node experienced periods of unresponsiveness during cluster bring-up.

**Symptoms:** Observed NIC-related migration instability in one period and later thermal symptoms in another.

**Investigation:** Reviewed BIOS/firmware, USB Ethernet adapter behavior, fan policy, and thermal conditions. Firmware changes improved the earlier migration-related behavior; later investigations also included cooling and hardware replacement planning.

**Resolution/status:** The node was subsequently reported restored to operation. Available notes do not establish one verified root cause covering every symptom, so no such claim is made here.

**Validation limit:** The project has verified cluster operation at later checkpoints, but a single comprehensive root-cause report and post-repair stress-test result have not been published.

**Lesson:** Separate symptoms into incidents and document evidence rather than collapsing them into an unproven explanation.

## Approach used across incidents

The repeatable pattern is **observe → isolate → change one variable → validate → document**. Operational recovery commands and detailed system identifiers are kept in the private runbook rather than included in this public summary.
