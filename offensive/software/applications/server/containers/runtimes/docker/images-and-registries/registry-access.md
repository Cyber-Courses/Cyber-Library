---
title: "Registry access: authenticating to and pulling private images"
description: "Reaching a private container registry with recovered or weak credentials, including cloud registry tokens and docker config files, to pull private images and read the code, configuration, and secrets they contain."
keywords:
  - registry access
  - docker login
  - pull private image
  - registry credentials
  - ECR GCR ACR
---

# Registry access

Private registries gate pulls behind credentials, which are routinely recoverable: a `~/.docker/config.json`, a Kubernetes image pull secret, or a cloud registry token from instance metadata. With them, pull any image the identity can read.

```bash
# Credentials from a recovered docker config (base64 auth entries)
cat ~/.docker/config.json | jq '.auths'

# Cloud registries mint short-lived tokens from the instance/workload identity
aws ecr get-login-password | docker login --username AWS --password-stdin <acct>.dkr.ecr.<region>.amazonaws.com
docker pull <acct>.dkr.ecr.<region>.amazonaws.com/<repo>:<tag>
```

## Exploitation notes

- A cloud identity on a compromised host or pod often has registry pull (or push) rights; mint the token from metadata rather than hunting for static creds.
- Pull rights alone leak source, configs, and secrets; push rights enable [Image backdooring](image-backdooring.md).
- Image pull secrets stored in orchestrators are a prime source, see [Image pull secret theft](../../containerd-and-cri-o/image-pull-secret-theft.md).

## References

- [Docker: configure registry credentials](https://docs.docker.com/reference/cli/docker/login/)
- [OCI distribution specification](https://github.com/opencontainers/distribution-spec)
