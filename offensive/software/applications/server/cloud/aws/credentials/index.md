---
title: "AWS credentials"
description: "Harvesting AWS credentials: long-term access keys, STS session tokens, the EC2 and ECS instance metadata service (IMDSv1/v2 and SSRF), Secrets Manager, Parameter Store, and KMS, and the service credential brokers that mint them."
keywords:
  - AWS credentials
  - access keys
  - STS
  - IMDS
  - Secrets Manager
  - Parameter Store
  - KMS
---

# Credentials

Every AWS API call needs a credential, so obtaining one is the first move and harvesting more is how an engagement widens. Credentials arrive as long-lived **access keys**, temporary **STS session tokens** (an access key, secret, session token, and expiry), and role credentials delivered through the **instance metadata service**. Each has different theft and reuse characteristics, and several AWS services hand back fresh credentials when read.

## What folds in here

- **[Access keys](access-keys.md)**: long-term `AKIA` pairs in files, environment variables, CI config, and source history.
- **[Instance metadata](instance-metadata/index.md)**: role credentials from IMDS on EC2 and ECS, including the IMDSv2 token flow and SSRF retrieval.
- **[STS tokens](sts-tokens.md)**: capturing and replaying short-lived session tokens.
- **[Secret stores](secret-stores/index.md)**: Secrets Manager, SSM Parameter Store, and KMS as credential sources.
- **[Credential brokers](credential-brokers.md)**: services that mint credentials for other principals.

The IMDS pages are the AWS end of the web [server-side request forgery](../../../web/code/injection/request-forgery/index.md) technique, cross-referenced rather than duplicated.

## References

- [HackTricks Cloud: AWS metadata and credentials](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/index.html)
- [AWS: EC2 instance metadata service (IMDSv2)](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/configuring-instance-metadata-service.html)
