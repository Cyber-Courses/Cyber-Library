---
title: "AMI: secrets in shared and public machine images"
description: "Finding secrets in shared or public AMIs and launching instances from them to inherit baked-in access."
keywords:
  - AMI
  - machine image
  - public
  - secrets
  - launch
---

# AMI

An **Amazon Machine Image** is a packaged root volume plus launch metadata. Teams bake credentials, keys, and application secrets into golden AMIs and then share them too broadly, so a shared or public AMI is both a data leak and a way to launch a host that already trusts attacker-staged access.

## Finding and using AMIs

```bash
# AMIs an account owns, and ones shared with you
aws ec2 describe-images --owners <acct>
aws ec2 describe-images --executable-users self

# Launch from a shared AMI, or make a volume from its snapshot to read offline
aws ec2 run-instances --image-id ami-xxxx --instance-type t3.micro --key-name <k>
```

An AMI references EBS snapshots; mounting those snapshots (see [snapshots](snapshots.md)) reads the baked-in filesystem without booting the image.

## Exploitation notes

- Public AMIs (`--executable-users all`) are enumerable region-wide and sometimes still carry live secrets from mistaken publication.
- Launching the image inherits any SSH keys, cron jobs, or agents baked in, so a poisoned or stale golden image can hand you persistence.
- Combine with [Image Builder](../image-builder.md): poisoning the pipeline that produces an AMI scales the compromise to every host launched from it.

## Tools

- **AWS CLI** (`describe-images`, `run-instances`): discovery and launch.
- **dsnap**: read the AMI's backing snapshot offline.

## References

- [HackTricks Cloud: AMIs](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-services/aws-ec2-ebs-elb-ssm-vpc-and-vpn-enum/index.html)
- [AWS: describe-images](https://docs.aws.amazon.com/cli/latest/reference/ec2/describe-images.html)
- [Hacking the Cloud: AWS](https://hackingthe.cloud/)
- [Stratus Red Team: AWS techniques](https://stratus-red-team.cloud/attack-techniques/AWS/)
- [Pacu (Rhino Security Labs)](https://github.com/RhinoSecurityLabs/pacu)
