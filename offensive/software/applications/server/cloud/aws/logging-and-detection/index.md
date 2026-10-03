---
title: "AWS logging and detection"
description: "Blinding AWS detection: disabling, diverting, and evading CloudTrail, GuardDuty, Config, and CloudWatch to suppress logging and alerting."
keywords:
  - CloudTrail
  - GuardDuty
  - AWS Config
  - CloudWatch
  - log tampering
  - detection evasion
  - anti-forensics
---

# Logging and detection

The audit plane is a target, not a constraint. Before noisy actions, or to erase their trace afterward, the account's logging and detection services are stopped, narrowed, diverted, or deleted. AWS centralizes this in four services: **CloudTrail** records API calls, **Config** records resource state, **GuardDuty** scores behavior for threats, and **CloudWatch** holds logs, metrics, and the alarms that page a responder. Degrading any one of them widens what can be done unseen.

The quietest moves are selective (narrow a trail's event selectors, archive a finding class) rather than destructive (delete the trail), because the stop and delete calls are themselves recorded, so they are only clean when they land before the record reaches somewhere you do not control.

## What folds in here

- **[CloudTrail](cloudtrail.md)**: stopping, deleting, and diverting trails, and dropping management and data events.
- **[GuardDuty](guardduty.md)**: disabling detectors and auto-archiving findings, and staying inside the analytics' blind spots.
- **[Config](config.md)**: stopping configuration recorders and deleting rules and delivery channels.
- **[CloudWatch](cloudwatch.md)**: deleting log groups, metric filters, and alarms, and disabling alarm actions.

## References

- [HackTricks Cloud: AWS defense evasion](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-defense-evasion/index.html)
- [Stratus Red Team: AWS defense evasion](https://stratus-red-team.cloud/attack-techniques/AWS/)
- [AWS: CloudTrail concepts](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-concepts.html)
