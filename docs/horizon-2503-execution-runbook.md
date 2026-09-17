# Horizon 8 2503 Subscription Execution Runbook

> **Change type:** production maintenance
> **Target:** Omnissa Horizon 8 2503 Subscription (8.15.0, build 14365030791)
> **UAG target:** a separately selected version confirmed compatible with both the current and target Horizon versions.
> **Out of scope:** vCenter/ESXi upgrade, certificate redesign, DNS renaming, and non-Horizon platform changes.

## 1. Change control

| Role | Responsibility |
| --- | --- |
| Change owner | Approves gates, owns the schedule, and calls stop/go decisions. |
| Horizon administrator | Upgrades Connection Servers and Agents; verifies Horizon health. |
| Network/UAG administrator | Deploys and validates the replacement UAG and external publishing path. |
| vSphere administrator | Verifies vCenter backup/recovery and VM provisioning. |
| DBA | Verifies the Events database backup and restore evidence. |
| Test owner | Executes end-user validation and signs off the pilot. |

Define the maintenance window, stakeholder notice, bridge details, escalation contacts, and a named decision-maker before proceeding.

## 2. Mandatory entry gates

All gates must be green before starting production changes.

| Gate | Evidence required | Owner |
| --- | --- | --- |
| Compatibility | Documented source-to-target compatibility for Connection Server, Agent, UAG, Connection Server OS, vCenter, and ESXi. | Horizon admin |
| Subscription license | Valid Omnissa subscription license activated before the server upgrade. Horizon 2503 has no 15-day subscription grace period. | Change owner |
| Baseline | Dated read-only inventory: servers, pools, agents, farms/add-ons, vCenter/ESXi, certificates, and current health. | Horizon/vSphere admins |
| Horizon health | Connection Server replication healthy; no unresolved critical alarms; vCenter connectivity and provisioning confirmed. | Horizon admin |
| Recovery | Recent, recoverable VCSA file backup; Events DB full backup plus restore verification; UAG JSON/INI export; separately retained certificates, keys, Keytabs, and shared secrets. | vSphere/DBA/network admins |
| Software | Approved Horizon and UAG installers, SHA-256 checksums, licensing/entitlement, and release-note review. | Change owner |
| Capacity | Replacement UAG resources, network, DNS/load-balancer, firewall, and certificate plan approved. | Network/UAG admin |
| Pilot | One available desktop selected with a documented recovery method and a test user. | Horizon/test owners |

**Stop condition:** a missing backup, failed restore verification, unknown compatibility, unhealthy replication, or unapproved external publishing change.

## 3. Pre-change procedure

1. Announce the start of the window and freeze Horizon configuration changes.
2. Capture final health evidence: Connection Server replication, pool state, desktop state, vCenter connectivity, provisioning state, and UAG health.
3. Confirm all backup artifacts are dated, stored in approved private storage, and recoverable by their designated owners.
4. Verify that no critical desktop session or business process will be interrupted; obtain business approval if a disconnect is necessary.
5. Confirm the Omnissa subscription license is active, then download the Horizon 2503 installers from Omnissa, verify SHA-256 checksums, and stage them on approved administration hosts.
6. Document the currently published UAG path and do not remove the existing appliance.

## 4. Connection Server upgrade

Upgrade one Connection Server at a time; keep the peer available whenever the documented procedure permits.

1. Place the first server into maintenance according to the current Omnissa upgrade guide. Drain/disable new connections as appropriate.
2. Run the approved Horizon 8 2503 Connection Server installer as an administrator, preserving the existing AD LDS instance and configuration.
3. Reboot only if required by the installer.
4. Validate the upgraded server before touching the peer:
   - service health and version;
   - AD LDS/Connection Server replication;
   - Horizon administrative access;
   - vCenter connectivity;
   - pool visibility and a basic internal launch test.
5. If validation passes, repeat for the remaining Connection Server.
6. Confirm all Connection Servers show the intended version and replication is healthy.

**Stop condition:** installer failure, degraded replication, lost vCenter connectivity, or a failed internal launch. Do not attempt an in-place downgrade; preserve evidence and follow the approved Horizon recovery procedure.

## 5. UAG replacement and cutover

1. Deploy a new, compatible UAG appliance using the approved template and the exported settings as a reference.
2. Restore/configure only through approved methods. Re-enter secrets, Keytabs, and private key material from the secrets system; do not place them in exports or source control.
3. Validate the new UAG on a non-production or temporary publishing path:
   - appliance health;
   - TLS certificate and chain;
   - Connection Server reachability;
   - authentication;
   - Blast/HTML Access and any enabled protocols;
   - MFA, RADIUS, SAML, or other integrated controls.
4. Change the load balancer/DNS/publication path to the new UAG during the approved cutover.
5. Test external user access from an appropriate network.
6. Retain the former UAG powered on and recoverable until the entire change is accepted.

**Rollback:** return the published path to the prior known-good UAG after confirming it is healthy. Do not delete it during the window.

## 6. Horizon Agent pilot and rollout

1. Select one available, representative desktop. Record its current agent version and recovery method.
2. Install the Horizon 8 2503 Agent using the approved feature set and reboot if required.
3. Validate with the test owner:
   - client login and desktop launch;
   - Blast and every enabled protocol;
   - reconnect after intentional disconnect;
   - HTML Access, if enabled;
   - dedicated assignment behavior;
   - peripherals, printing, USB redirection, smart card, and MFA as applicable;
   - image/provisioning workflow as applicable.
4. Review Connection Server events and desktop agent health.
5. Obtain explicit pilot sign-off before upgrading the remaining agents in controlled batches.
6. Stop the batch immediately if failures appear; retain enough known-good desktops for service continuity.

## 7. Final validation and acceptance

| Test | Acceptance result |
| --- | --- |
| Connection Server versions | Every server reports the planned 2503 build. |
| Replication | Healthy across all Connection Servers. |
| Internal access | Authentication and desktop launch succeed. |
| External access | Authentication and desktop launch succeed through the new UAG. |
| Session behavior | Reconnect and enabled protocols work. |
| Pool operations | Assignment, entitlement, and provisioning behave as expected. |
| Integrations | vCenter connectivity and all enabled add-ons/identity services pass their defined tests. |
| Monitoring | No unresolved critical Horizon, UAG, vCenter, or agent alerts. |

## 8. Rollback principles

- **Connection Servers:** stop at the failed server; do not perform an in-place downgrade or casually revert a replica VM snapshot. Escalate to the documented Horizon recovery procedure with the verified recovery artifacts.
- **UAG:** restore external publishing to the retained prior UAG if the replacement fails validation.
- **Agent pilot:** use the documented desktop recovery method; do not continue agent rollout without pilot sign-off.
- **Data:** restoration decisions for vCenter or the Events database require their named recovery owners and the pre-verified recovery plan.

## 9. Closeout

1. Record final versions, installer checksums, timestamps, approvers, and validation results in the private change record.
2. Retain recovery artifacts according to the organization’s backup and records policy.
3. Remove temporary publishing paths only after the agreed stability period.
4. Schedule separate follow-up work for certificate/DNS cleanup, vSphere lifecycle modernization, and any deferred issues.
