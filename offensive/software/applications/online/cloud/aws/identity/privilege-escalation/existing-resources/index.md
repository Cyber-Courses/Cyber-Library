---
title: "Existing resources"
order: 5
description: "Escalating through resources that already run with a privileged role: updating CloudFormation stacks, Glue dev endpoints, and Lambda code, or minting presigned URLs and EC2 Instance Connect access."
keywords:
  - UpdateStack
  - UpdateFunctionCode
  - dev endpoint
  - presigned URL
  - EC2 Instance Connect
  - existing resource
---

# Existing resources

When a resource already runs with a privileged role, you do not need `iam:PassRole`: you only need permission to **change what that resource runs**. Overwriting its code or configuration makes the existing role execute your payload.

## Paths

- **[UpdateStack](update-stack.md)**: change a CloudFormation stack that carries a privileged stack role.
- **[UpdateDevEndpoint](update-dev-endpoint.md)**: push an SSH key to a Glue dev endpoint running a privileged role.
- **[UpdateFunctionCode](update-function-code.md)**: overwrite a Lambda that already runs a privileged execution role.
- **[Presigned URL](presigned-url.md)**: mint time-limited signed access to resources under your own permissions.
- **[Instance Connect](instance-connect.md)**: push a temporary SSH key to an instance carrying a privileged profile.
- **[AssociateInstanceProfile](associate-instance-profile.md)**: attach a privileged instance profile to an instance you already control.

## References

- [Rhino Security Labs: AWS privilege escalation (resource update paths)](https://rhinosecuritylabs.com/aws/aws-privilege-escalation-methods-mitigation/)
- [BishopFox: iam-vulnerable](https://github.com/BishopFox/iam-vulnerable)
- [Pacu (Rhino Security Labs)](https://github.com/RhinoSecurityLabs/pacu)
- [Hacking the Cloud: AWS](https://hackingthe.cloud/)
