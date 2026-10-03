---
title: "AWS credentials"
description: "Where AWS credentials come from and how they are abused: long-lived access keys, temporary STS session tokens, the EC2 instance metadata service (IMDS), environment variables, and the secret stores that hand back more credentials."
keywords:
  - access keys
  - STS token
  - IMDS
  - Secrets Manager
  - SSM Parameter Store
---

# Credentials

Every AWS action needs a credential, so obtaining one is the first move and harvesting more is how an engagement widens. Credentials come as long-lived **access keys**, temporary **STS session tokens** (with a session token and expiry), and role credentials delivered through the **instance metadata service**. Each has different theft and reuse characteristics.

The pages here cover recovering role credentials from **IMDS** (including the IMDSv2 token-request flow, reached through an [SSRF](../../../web/code/injection/request-forgery/index.md) on an instance), finding access keys in environment variables, files, and source, reading **Secrets Manager** and **SSM Parameter Store** as credential sources, and understanding STS token scope and expiry for reuse.

## What folds in here

- **Secret stores** (Secrets Manager, SSM Parameter Store, KMS-wrapped material) are treated as credential sources here rather than a separate surface.
- The IMDS page is the AWS end of the web [server-side request forgery](../../../web/code/injection/request-forgery/index.md) technique, cross-referenced rather than duplicated.

## References

- [HackTricks Cloud: AWS metadata and credentials](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/index.html)
- [AWS: EC2 instance metadata service (IMDSv2)](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/configuring-instance-metadata-service.html)
