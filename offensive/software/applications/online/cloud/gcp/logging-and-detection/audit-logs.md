---
title: "Audit Logs: disabling data-access logging"
description: "Disabling or narrowing Cloud Audit Logs, especially data-access logs, to hide API calls."
keywords:
  - Cloud Audit Logs
  - data access logs
  - audit config
  - logging
  - detection evasion
  - anti-forensics
---

# Audit Logs

Cloud Audit Logs have three streams: **Admin Activity** (always on, cannot be disabled), **Data Access** (off by default except for a few services, and the one that records reads and data-plane calls), and **System Event**. The data-access configuration lives in the IAM policy's `auditConfigs`, so a principal that can set the IAM policy can turn data-access logging off and hide the bulk of read activity.

## Read the current audit config

```bash
gcloud projects get-iam-policy <project> --format=json > policy.json
# the "auditConfigs" block lists which services log DATA_READ / DATA_WRITE
```

## Disable data-access logging

```bash
# remove or empty the auditConfigs block in policy.json, then write it back
gcloud projects set-iam-policy <project> policy.json
```

Setting `auditConfigs` to empty (or dropping `DATA_READ`/`DATA_WRITE` for the services you will touch) stops those reads from being recorded, while Admin Activity keeps logging control-plane writes.

## Exploitation notes

- **Admin Activity logs cannot be disabled**, so the `set-iam-policy` call that disables data-access logging is itself recorded; this is a trade of one loud write for silence on all subsequent reads.
- Audit config can be set at the **organization, folder, or project** level; an org-level enablement overrides a project attempt, so check the resource hierarchy before assuming reads are dark.
- Data-access logs are already off by default in most projects, so in many targets the read activity is unlogged without any action.

## Tools

- **gcloud** (`projects get-iam-policy` / `set-iam-policy`): read and rewrite the audit config.
- **ScoutSuite** / **Prowler**: report which services have data-access logging enabled.

## References

- [HackTricks Cloud: GCP](https://cloud.hacktricks.wiki/en/pentesting-cloud/gcp-security/index.html)
- [Hacking the Cloud: GCP](https://hackingthe.cloud/)
- [Google: configure Data Access audit logs](https://cloud.google.com/logging/docs/audit/configure-data-access)
