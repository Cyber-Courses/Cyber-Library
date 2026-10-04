---
title: "AWS"
description: "Offensive techniques against Amazon Web Services, broken down by surface: identity, credentials, and the compute, storage, serverless, data, networking, logging, and messaging classes, with enumeration folded into each. Attacks center on the IAM graph and the AWS API, reached with keys or roles rather than host exploits."
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

- **[Identity](identity/index.md)**: the IAM graph, role assumption and `PassRole`, policy abuse, federation, and the privilege-escalation paths, plus the Cognito, Identity Center, and Organizations identity layers.
- **[Credentials](credentials/index.md)**: access keys, STS session tokens, instance metadata (IMDS), the secret stores, and the service credential brokers.
- **[Compute](compute/index.md)**: EC2 user data and instance profiles, SSM, containers and ECR, and the Beanstalk, Image Builder, App Runner, Batch, and Lightsail runtimes.
- **[Storage](storage/index.md)**: S3 bucket and object exposure, EBS and EFS recovery, FSx and Storage Gateway, and AWS Backup.
- **[Serverless](serverless/index.md)**: Lambda execution roles and code, API Gateway, Function URLs, Step Functions, and EventBridge.
- **[Data](data/index.md)**: RDS, DynamoDB, Redshift, SageMaker, Glue, Athena, Lake Formation, and the other managed data stores.
- **[Networking](networking/index.md)**: security groups and VPC reachability, Route53 and CloudFront, dangling-DNS takeover, and the edge services.
- **[Logging and detection](logging-and-detection/index.md)**: disabling, diverting, and evading CloudTrail, GuardDuty, Config, and CloudWatch.
- **[Messaging](messaging/index.md)**: SES mail sending and phishing, and SNS and SQS topic and queue abuse.

## References

- [HackTricks Cloud: AWS](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/index.html)
- [Rhino Security Labs: AWS IAM privilege escalation methods](https://rhinosecuritylabs.com/aws/aws-privilege-escalation-methods-mitigation/)
- [BishopFox: iam-vulnerable](https://github.com/BishopFox/iam-vulnerable)
- [PayloadsAllTheThings: AWS](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Cloud%20-%20AWS%20Pentest.md)
