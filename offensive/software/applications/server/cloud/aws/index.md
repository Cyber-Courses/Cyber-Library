---
title: "AWS"
description: "Offensive techniques against Amazon Web Services, broken down by surface: identity, credentials, and the compute, storage, serverless, data, and networking resource classes, with enumeration folded into each. Attacks center on the IAM graph and the AWS API, reached with keys or roles rather than host exploits."
keywords:
  - AWS
  - IAM
  - IMDS
  - S3
  - privilege escalation
---

# AWS

Amazon Web Services is driven entirely through its **API**, authenticated with IAM **access keys** or temporary **role credentials**. Nearly every attack is an IAM question: what principal do I hold, what can it do, what can it escalate to, and which resources can it reach. The surfaces below follow that shape, with **privilege escalation** living inside [identity](identity/index.md) and data theft inside [storage](storage/index.md) and [data](data/index.md).

## Enumeration

Enumeration is not a separate surface: each surface below covers enumerating its own resources, and unauthenticated discovery (public buckets, exposed services, account IDs) lives in the surface it targets. What spans surfaces is the **account-wide inventory**, run first with **ScoutSuite** and **Prowler**, the IAM graph mapped with **Pacu** and **PMapper**, and the current principal's rights established with `aws sts get-caller-identity` and permission enumeration. Those feed every surface below.

## Surfaces

- **[Identity](identity/index.md)**: the IAM graph, role assumption and `PassRole`, policy abuse, and privilege-escalation paths.
- **[Credentials](credentials/index.md)**: access keys, STS session tokens, instance metadata (IMDS), and secret stores as credential sources.
- **[Compute](compute/index.md)**: EC2, ECS, and EKS abuse, user-data, and the run-command and snapshot paths.
- **[Storage](storage/index.md)**: S3 bucket and object exposure, and EBS snapshots.
- **[Serverless](serverless/index.md)**: Lambda function and layer abuse through its execution role.
- **[Data](data/index.md)**: RDS, DynamoDB, and the managed data stores.
- **[Networking](networking/index.md)**: security groups, exposed services, and VPC reachability.

## References

- [HackTricks Cloud: AWS](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/index.html)
- [Rhino Security Labs: AWS IAM privilege escalation methods](https://rhinosecuritylabs.com/aws/aws-privilege-escalation-methods-mitigation/)
- [PayloadsAllTheThings: AWS](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Cloud%20-%20AWS%20Pentest.md)
