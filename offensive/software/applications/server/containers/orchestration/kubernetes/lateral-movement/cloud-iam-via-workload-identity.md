---
title: "Cloud IAM via workload identity: pivoting from a pod into the cloud"
description: "Pivoting from a Kubernetes pod into the cloud account through its mapped workload identity, such as AWS IRSA, GKE Workload Identity, or Azure workload identity, which grants the pod a cloud role whose permissions extend well beyond the cluster."
keywords:
  - workload identity
  - IRSA
  - GKE workload identity
  - cloud pivot
  - IAM
---

# Cloud IAM via workload identity

Managed clusters map pods to cloud identities so workloads can call cloud APIs without static keys: AWS IRSA, GKE Workload Identity, and Azure workload identity. A compromised pod inherits that cloud role, and those roles are frequently broad (read buckets, manage resources, read secrets managers), giving a pivot out of the cluster into the account.

```bash
# The pod carries a projected token exchanged for cloud credentials
env | grep -iE 'AWS_ROLE_ARN|AWS_WEB_IDENTITY_TOKEN_FILE|AZURE_|GOOGLE_'
aws sts get-caller-identity                       # with IRSA env present
aws s3 ls; aws secretsmanager list-secrets        # exercise the role
```

## Exploitation notes

- The pod's own identity is often more useful than the node's, and avoids the metadata endpoint entirely; check for the workload-identity environment first.
- Cloud roles mapped to pods are commonly over-scoped; enumerate the role's policies, then reach secrets managers and storage.
- Where no workload identity is mapped, fall back to the node role through [Cloud metadata from pod](../cluster-enumeration/cloud-metadata-from-pod.md).

## References

- [AWS: IAM roles for service accounts](https://docs.aws.amazon.com/eks/latest/userguide/iam-roles-for-service-accounts.html)
- [GKE: Workload Identity](https://cloud.google.com/kubernetes-engine/docs/concepts/workload-identity)
