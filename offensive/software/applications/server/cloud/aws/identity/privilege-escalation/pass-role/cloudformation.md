---
title: "CloudFormation: PassRole into a stack service role"
description: "iam:PassRole into a CloudFormation stack to create resources and run actions under a privileged stack role."
keywords:
  - PassRole
  - CloudFormation
  - stack role
  - CreateStack
  - deployment
---

# CloudFormation

CloudFormation can deploy a stack using a **service role** rather than your own permissions. With `iam:PassRole` and `cloudformation:CreateStack`, you pass a privileged role and submit a template that provisions whatever that role can create, including new IAM principals or policies.

## Deploy with a passed role

```bash
# template.yaml creates, for example, an admin user or attaches a policy
aws cloudformation create-stack --stack-name x \
  --template-body file://template.yaml \
  --role-arn arn:aws:iam::<acct>:role/<privileged-stack-role> \
  --capabilities CAPABILITY_NAMED_IAM
```

Because the stack role performs the resource creation, a template that creates an `AWS::IAM::User` with an access key, or attaches `AdministratorAccess`, succeeds even though your own principal cannot.

## Exploitation notes

- `CAPABILITY_NAMED_IAM` is required when the template touches IAM; the role, not you, must hold the IAM permissions.
- Updating an existing stack that already carries a privileged role is the [existing-resources](../existing-resources/update-stack.md) variant and needs no `PassRole`.
- Stack outputs can return the created access key directly, so no separate retrieval step is needed.

## Tools

- **AWS CLI** (`cloudformation create-stack`).
- **Pacu**: CloudFormation privesc and data modules.

## References

- [Rhino Security Labs: AWS privilege escalation (CloudFormation)](https://rhinosecuritylabs.com/aws/aws-privilege-escalation-methods-mitigation/)
- [BishopFox: iam-vulnerable](https://github.com/BishopFox/iam-vulnerable)
- [Pacu (Rhino Security Labs)](https://github.com/RhinoSecurityLabs/pacu)
- [Hacking the Cloud: AWS](https://hackingthe.cloud/)
