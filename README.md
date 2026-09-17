# Omnissa Horizon 8 2606 Upgrade Runbook

A public-safe execution plan for upgrading a Horizon 8 environment to Horizon 8 2606. It intentionally excludes topology, hostnames, IP addresses, certificate data, credentials, backup exports, and collected inventory.

## Scope

- Horizon Connection Servers: upgrade sequentially.
- Unified Access Gateway (UAG): deploy a replacement appliance and cut over only after validation.
- Horizon Agents: upgrade one pilot desktop before the broader rollout.
- vCenter and ESXi: remain unchanged by this Horizon change.

## Before using this runbook

1. Confirm the exact source and target versions in the current [Omnissa interoperability matrix](https://docs.omnissa.com/Horizon8InstallUpgrade/CompatibilityMatrixforVariousVersionsofHorizon8Components).
2. Read the relevant Horizon 8 2606 release notes and installation/upgrade documentation.
3. Replace every placeholder in the runbook with approved, environment-specific information in a private change record.
4. Do not store exports, passwords, private keys, installers, database backups, or inventory results in this public repository.

## Execution plan

See [the detailed runbook](docs/horizon-2606-execution-runbook.md). It includes change gates, responsibilities, ordered procedures, validation, rollback decisions, and a closeout checklist.

## Safety model

This is a staged upgrade, not an in-place rollback exercise:

- Do not attempt an in-place Connection Server downgrade.
- Retain the existing UAG until the replacement has passed external validation.
- Stop the rollout after any failed validation gate.
- Keep desktop-agent rollout limited to the pilot until it is signed off.
