---
title: "Node identity: kubelet and managed identity through IMDS"
description: "Abusing an AKS node pool's kubelet and managed identity through IMDS to escalate from a pod to the subscription."
keywords:
  - AKS node
  - kubelet
  - managed identity
  - IMDS
  - node pool
---

# Node identity

An AKS node pool runs on VMs that carry a **kubelet managed identity** and the node's own identity. A pod that can reach the node's instance metadata endpoint (often not blocked) mints that identity's Azure token and escalates out of the cluster into the subscription.

## From a pod to an Azure token

```bash
# inside a pod on the node, hit IMDS for the node/kubelet identity token
curl -s -H Metadata:true \
 "http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https://management.azure.com/"
```

## Exploitation notes

- Blocking pod access to `169.254.169.254` is a frequently missed control; where it is missed, any pod RCE becomes an Azure identity.
- The kubelet identity typically holds `AcrPull` and node-management rights, a lead into [Container Registry](../container-registry/index.md) and the node resource group.
- Workload-identity (federated) pods assume a role through OIDC instead; read the projected token and exchange it rather than hitting IMDS.

## Tools

- **curl** from a compromised pod; **Peirates** automates node and cloud-identity pivots.
- **Azure CLI** once the token is exported.

## References

- [Microsoft: AKS managed identities](https://learn.microsoft.com/azure/aks/use-managed-identity)
- [HackTricks Cloud: AKS node identity](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
- [Peirates](https://github.com/inguardians/peirates)
