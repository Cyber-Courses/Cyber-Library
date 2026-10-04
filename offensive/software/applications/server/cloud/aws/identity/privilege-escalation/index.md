---
title: "Privilege escalation"
description: "The full catalog of AWS IAM privilege-escalation paths: iam:PassRole into a service, policy and credential manipulation, trust-policy and existing-resource rewrites, and group-membership abuse."
keywords:
  - IAM privilege escalation
  - PassRole
  - iam-vulnerable
  - policy manipulation
  - trust policy
---

# Privilege escalation

AWS privilege escalation is a permission problem. A principal that holds one of a known set of dangerous actions can promote itself to administrator without touching a host. The catalog below is the union of the BishopFox **iam-vulnerable** paths and the Rhino Security Labs method list, grouped by the primitive each one abuses.

## The paths

- **[PassRole](pass-role/index.md)**: `iam:PassRole` combined with a service-creation action (EC2, Lambda, Glue, CloudFormation, Data Pipeline, CodeBuild, SageMaker) to run code as a privileged role.
- **[Policy manipulation](policy-manipulation/index.md)**: editing IAM policy directly, through new default policy versions and attaching or inlining user, group, and role policies.
- **[Credential creation](credential-creation/index.md)**: minting new credentials for a principal you can write to, through access keys and console login profiles.
- **[Existing resources](existing-resources/index.md)**: escalating through resources that already run with a privileged role, by updating stacks, dev endpoints, and function code, or minting presigned URLs and Instance Connect access.
- **[Group membership](group-membership.md)**: `iam:AddUserToGroup` to join a group that carries more permissions.
- **[Trust policy](trust-policy.md)**: rewriting a role's `AssumeRolePolicyDocument` so you can assume it.

## Choosing a path

The quietest paths require no new resource: a `CreatePolicyVersion` or `AttachUserPolicy` changes the principal in place. Paths that create a resource (an EC2 instance, a Lambda) are louder but work when the only lever you hold is `PassRole` plus a run action. PMapper and Pacu's `iam__privesc_scan` enumerate which of these the current principal can reach.

## References

- [BishopFox: iam-vulnerable](https://github.com/BishopFox/iam-vulnerable)
- [Rhino Security Labs: 21 ways to escalate privileges in AWS](https://rhinosecuritylabs.com/aws/aws-privilege-escalation-methods-mitigation/)
- [HackTricks Cloud: AWS privilege escalation](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-privilege-escalation/index.html)
- [Pacu (Rhino Security Labs)](https://github.com/RhinoSecurityLabs/pacu)
- [Hacking the Cloud: AWS](https://hackingthe.cloud/)
