---
title: "Repository policy: cross-account pull and push"
order: 2
description: "Abusing a permissive ECR repository policy to pull or push images across account boundaries."
keywords:
  - ECR
  - repository policy
  - cross-account
  - push
  - pull
---

# Repository policy

An ECR **repository policy** is a resource policy that can grant other principals, including whole external accounts, the right to pull or push. A policy that allows `ecr:*` or names a broad principal lets an attacker in one account read images from, or push images into, a repository in another, turning a loose policy into cross-account data access or a supply-chain write.

## Reading and abusing the policy

```bash
aws ecr get-repository-policy --repository-name <r> --query policyText --output text

# If pull is allowed cross-account: authenticate to the owning registry and pull
aws ecr get-login-password --region <region> | docker login --username AWS \
  --password-stdin <owner-acct>.dkr.ecr.<region>.amazonaws.com
docker pull <owner-acct>.dkr.ecr.<region>.amazonaws.com/<r>:<tag>
```

If push is allowed, the repository is an [image poisoning](image-poisoning.md) target.

## Exploitation notes

- A `Principal` of `*` or another account root on a pull action exposes every image in the repository cross-account.
- Push rights cross-account are the dangerous case: they let you plant a tag that the owner's deploy pipeline will run.
- Combine with [enumeration](../../identity/enumeration.md) to find which repositories the policies actually expose, then target the ones feeding privileged workloads.

## Tools

- **AWS CLI** (`get-repository-policy`, `ecr get-login-password`): read the policy, authenticate.
- **docker** / **crane**: pull or push across the boundary.

## References

- [AWS: ECR repository policies](https://docs.aws.amazon.com/AmazonECR/latest/userguide/repository-policies.html)
- [HackTricks Cloud: ECR](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-services/aws-ecr-enum.html)
- [Hacking the Cloud: AWS](https://hackingthe.cloud/)
- [Stratus Red Team: AWS techniques](https://stratus-red-team.cloud/attack-techniques/AWS/)
- [CloudFox (Bishop Fox)](https://github.com/BishopFox/cloudfox)
