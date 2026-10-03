---
title: "SCP manipulation: lifting organization-wide guardrails"
description: "Editing service control policies from the management account to lift guardrails across the organization."
keywords:
  - SCP
  - service control policy
  - Organizations
  - management account
  - guardrail
---

# SCP manipulation

Service control policies are the organization-wide permission ceiling: they cap what principals in member accounts may do, regardless of their IAM policies. From the management account (or a delegated admin), editing or detaching an SCP removes that ceiling, unblocking actions the org was relying on SCPs to deny.

## Loosen a guardrail

```bash
aws organizations list-policies --filter SERVICE_CONTROL_POLICY
aws organizations update-policy --policy-id <scp-id> \
  --content '{"Version":"2012-10-17","Statement":[{"Effect":"Allow","Action":"*","Resource":"*"}]}'
# or detach a restrictive SCP from an OU or account
aws organizations detach-policy --policy-id <scp-id> --target-id <ou-or-account>
```

## Exploitation notes

- SCPs do not grant permissions, they bound them, so lifting an SCP only matters alongside IAM permissions in the target account, but it removes a control teams assume is holding.
- This requires management-account access; it is a post-compromise move once you hold the org root, often paired with [member account role](member-account-role.md).
- Editing the FullAWSAccess baseline or attaching an allow-all policy is the broadest change and the most visible.

## Tools

- **AWS CLI** (`organizations update-policy` / `detach-policy`).

## References

- [AWS: service control policies](https://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_policies_scps.html)
- [HackTricks Cloud: AWS Organizations](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-services/aws-organizations-enum.html)
