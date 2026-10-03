---
title: "Security groups: finding internet-exposed ports and services"
description: "Mapping and abusing over-permissive security groups that expose management ports and internal services to the internet."
keywords:
  - security group
  - ingress
  - 0.0.0.0/0
  - exposure
  - port
---

# Security groups

A security group is a stateful allow-list of ingress and egress rules bound to an ENI. The common failure is an ingress rule with a `0.0.0.0/0` source on a management or database port, which puts SSH, RDP, or a database straight on the internet. With `ec2:Describe*` you read every group and resolve which instances they front, turning the account's network posture into a target list.

## Finding wide-open ingress

```bash
# every rule that allows the whole internet
aws ec2 describe-security-groups \
  --query "SecurityGroups[?IpPermissions[?contains(IpRanges[].CidrIp, '0.0.0.0/0')]].{Id:GroupId,Name:GroupName}"

# which ports are exposed on each group
aws ec2 describe-security-groups \
  --query "SecurityGroups[].IpPermissions[?contains(IpRanges[].CidrIp,'0.0.0.0/0')].[FromPort,ToPort]"
```

## Mapping groups to live hosts

```bash
# instances behind a group, with their public addresses
aws ec2 describe-instances \
  --filters Name=instance.group-id,Values=<sg-id> \
  --query "Reservations[].Instances[].[InstanceId,PublicIpAddress,PrivateIpAddress]"
```

## Exploitation notes

- `22`, `3389`, `5432`, `3306`, `6379`, `27017`, and `9200` open to `0.0.0.0/0` are the high-value finds: they front SSH, RDP, and unauthenticated or weakly authenticated data stores.
- Referenced-group rules (source is another security group, not a CIDR) only matter once you are inside the VPC, so pair this with [VPC](vpc.md) pivoting.
- Egress rules matter for your own exfiltration path off a compromised instance; a group that allows all egress is an open door outward.

## Tools

- **AWS CLI** (`ec2 describe-security-groups`, `describe-instances`): the enumeration.
- **ScoutSuite** / **Prowler**: flag internet-exposed ports account-wide.
- **nmap** / **masscan**: confirm reachability of the exposed ports from outside.

## References

- [HackTricks Cloud: AWS EC2 and VPC](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-services/aws-ec2-vpc-and-network-security.html)
- [AWS: security groups for your VPC](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-security-groups.html)
