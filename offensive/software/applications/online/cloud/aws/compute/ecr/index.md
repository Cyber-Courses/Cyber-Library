---
title: "ECR"
order: 4
description: "Attacking Elastic Container Registry: pulling from public or policy-exposed repositories and poisoning images that downstream compute will run."
keywords:
  - ECR
  - container registry
  - repository policy
  - image poisoning
  - public repository
---

# ECR

Elastic Container Registry stores the images that ECS, EKS, Lambda, and App Runner pull and run. That makes it a two-way target: images are a **source of secrets** (credentials and internal detail baked into layers), and a writable repository is a **supply-chain foothold**, because a poisoned tag runs on the next deploy under whatever role the workload holds.

## What folds in here

- **[Public repository](public-repository.md)**: pulling from public or loosely-scoped repositories to mine secrets.
- **[Repository policy](repository-policy.md)**: cross-account pull and push through a permissive repository policy.
- **[Image poisoning](image-poisoning.md)**: pushing a backdoored tag that downstream compute will run.

## Authenticating to ECR

```bash
aws ecr get-login-password | docker login --username AWS --password-stdin \
  <acct>.dkr.ecr.<region>.amazonaws.com
aws ecr describe-repositories ; aws ecr list-images --repository-name <r>
```

## References

- [HackTricks Cloud: ECR](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-services/aws-ecr-enum.html)
- [AWS: ECR repository policies](https://docs.aws.amazon.com/AmazonECR/latest/userguide/repository-policies.html)
- [Hacking the Cloud: AWS](https://hackingthe.cloud/)
- [Stratus Red Team: AWS techniques](https://stratus-red-team.cloud/attack-techniques/AWS/)
- [CloudFox (Bishop Fox)](https://github.com/BishopFox/cloudfox)
