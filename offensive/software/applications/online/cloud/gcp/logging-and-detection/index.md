---
title: "GCP logging and detection"
order: 8
description: "Blinding GCP detection: tampering with Cloud Logging sinks, disabling data-access audit logs, and weakening Security Command Center."
keywords:
  - Cloud Logging
  - audit logs
  - log sink
  - Security Command Center
  - detection evasion
  - anti-forensics
---

# Logging and detection

GCP records activity in two layers: **Cloud Audit Logs** capture the API calls, and **Cloud Logging** routes everything through log sinks to buckets and external destinations, with **Security Command Center** raising findings on top. An operator degrades these to act unseen, choosing between the blunt (delete a sink, disable a log type) and the quiet (an exclusion filter that drops only your events), weighed against what still records at the organization level above the project.

## What folds in here

- **[Cloud Logging](cloud-logging.md)**: deleting or diverting log sinks and adding exclusion filters to drop activity from the logs.
- **[Audit Logs](audit-logs.md)**: disabling or narrowing Cloud Audit Logs, especially the data-access logs, to hide API calls.
- **[Security Command Center](security-command-center.md)**: muting or disabling findings and sources to suppress alerting.

## References

- [HackTricks Cloud: GCP](https://cloud.hacktricks.wiki/en/pentesting-cloud/gcp-security/index.html)
- [Stratus Red Team: GCP techniques](https://stratus-red-team.cloud/attack-techniques/GCP/)
- [Hacking the Cloud: GCP](https://hackingthe.cloud/)
