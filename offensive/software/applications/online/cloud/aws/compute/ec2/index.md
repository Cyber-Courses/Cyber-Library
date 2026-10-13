---
title: "EC2"
order: 1
description: "Attacking EC2 instances: user-data secrets and code, the attached instance profile and its role, and data recovery from EBS snapshots and shared AMIs."
keywords:
  - EC2
  - user data
  - instance profile
  - snapshots
  - AMI
---

# EC2

EC2 is the richest compute surface in AWS because an instance ties together a **role** (the instance profile), a **boot script** (user data), and **persistent disks** (EBS volumes and the AMIs and snapshots they come from). Each is a separate way in: steal the role from the metadata service, read secrets left in user data, or recover another instance's data from a snapshot you can mount.

## What folds in here

- **[User data](user-data.md)**: reading boot scripts for secrets, or setting user data to run code on the next boot.
- **[Instance profile](instance-profile.md)**: harvesting the attached role's credentials from the metadata service.
- **[Snapshots](snapshots.md)**: creating, sharing, and mounting EBS snapshots to read volumes offline.
- **[AMI](ami.md)**: secrets baked into shared or public machine images, and launching from them.

Launching a *new* instance with a passed role is a privilege-escalation path and lives under [identity PassRole](../../identity/privilege-escalation/pass-role/ec2.md); the pages here cover instances that already exist.

## References

- [HackTricks Cloud: AWS EC2 enumeration and privesc](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-services/aws-ec2-ebs-elb-ssm-vpc-and-vpn-enum/index.html)
- [Rhino Security Labs: AWS privilege escalation methods](https://rhinosecuritylabs.com/aws/aws-privilege-escalation-methods-mitigation/)
- [Hacking the Cloud: AWS](https://hackingthe.cloud/)
- [Stratus Red Team: AWS techniques](https://stratus-red-team.cloud/attack-techniques/AWS/)
