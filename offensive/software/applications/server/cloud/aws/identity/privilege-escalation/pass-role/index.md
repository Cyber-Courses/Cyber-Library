---
title: "PassRole"
description: "iam:PassRole into a service that runs with the passed role: handing a privileged role to EC2, Lambda, Glue, CloudFormation, Data Pipeline, CodeBuild, or SageMaker."
keywords:
  - PassRole
  - service role
  - EC2
  - Lambda
  - CloudFormation
---

# PassRole

`iam:PassRole` is the permission to hand an IAM role to an AWS service. On its own it does nothing; combined with an action that **creates or updates a resource that runs with a role**, it lets you pass a role more privileged than your own to something that will execute your code or return its credentials. The target role only needs a trust policy allowing the service (for example `ec2.amazonaws.com`), which is the default when the role was built for that service.

The pattern is always: find a passable privileged role (`iam:ListRoles`, or the graph from [enumeration](../../enumeration.md)), then drive it through one of the service creations below.

## Targets

- **[EC2](ec2.md)**: launch an instance with a privileged instance profile, then read its credentials from IMDS.
- **[Lambda](lambda.md)**: create a function with a privileged execution role and invoke it.
- **[Glue](glue.md)**: a Glue job or dev endpoint running as a passed role.
- **[CloudFormation](cloudformation.md)**: deploy a stack with a passed service role.
- **[Data Pipeline](data-pipeline.md)**: a pipeline whose activities run as a passed role.
- **[CodeBuild](codebuild.md)**: a build project running as a passed service role.
- **[SageMaker](sagemaker.md)**: a notebook or training job running as a passed role.

## Finding passable roles

```bash
aws iam list-roles --query 'Roles[].Arn'
# a role is useful if its trust policy allows the service you can drive
aws iam get-role --role-name <r> --query 'Role.AssumeRolePolicyDocument'
```

## References

- [Rhino Security Labs: AWS privilege escalation (PassRole paths)](https://rhinosecuritylabs.com/aws/aws-privilege-escalation-methods-mitigation/)
- [HackTricks Cloud: iam:PassRole](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-privilege-escalation/aws-iam-privesc.html)
