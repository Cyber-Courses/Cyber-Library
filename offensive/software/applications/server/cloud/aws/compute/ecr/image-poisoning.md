---
title: "Image poisoning: backdooring a tag downstream compute will run"
description: "Pushing a backdoored image tag that ECS, EKS, or Lambda will pull and run under a privileged role."
keywords:
  - ECR
  - image poisoning
  - supply chain
  - tag
  - backdoor
---

# Image poisoning

If you can push to an ECR repository, you can overwrite a tag that running workloads pull. ECS services, EKS deployments, and container-image Lambdas re-pull on deploy or scale, so a backdoored `:latest` (or a pinned tag you can overwrite) runs your code under whatever role the workload holds, which is both execution and persistence.

## Poisoning a tag

```bash
aws ecr get-login-password | docker login --username AWS --password-stdin \
  <acct>.dkr.ecr.<region>.amazonaws.com

# Rebuild the real image with an added payload, keep the same tag
docker build -t <acct>.dkr.ecr.<region>.amazonaws.com/<repo>:latest .
docker push <acct>.dkr.ecr.<region>.amazonaws.com/<repo>:latest
```

Force the pull by triggering a new deployment (`ecs update-service --force-new-deployment`) where you have the rights, or wait for the next scale or release.

## Exploitation notes

- Keep the original entrypoint and add your payload alongside it, so the service still works and the change is not noticed from behavior alone.
- Tag immutability, when enabled, blocks overwriting an existing tag; push a new tag and wait for a deploy that references it instead.
- The payload inherits the task or function role, so pair with the task-role theft in [containers](../containers.md) to know what you will gain.

## Tools

- **docker** / **crane**: rebuild and push the poisoned image.
- **Pacu** (`ecs__backdoor_task_def`): redirect a task definition to your image.

## References

- [HackTricks Cloud: ECR image poisoning](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-services/aws-ecr-enum.html)
- [AWS: ECR image tag mutability](https://docs.aws.amazon.com/AmazonECR/latest/userguide/image-tag-mutability.html)
