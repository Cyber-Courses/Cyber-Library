---
title: "Unauthenticated registry access: anonymous pull and push"
description: "Abusing a container registry that allows anonymous access to pull private images without credentials, or, worse, to push, which lets an attacker overwrite a tag with a backdoored image that downstream systems then deploy."
keywords:
  - unauthenticated registry
  - anonymous pull
  - anonymous push
  - registry misconfiguration
  - image backdoor
---

# Unauthenticated registry access

Self-hosted registries frequently ship with no authentication, or with read or even write open to anonymous clients. Anonymous pull leaks every image; anonymous push is a supply-chain foothold, letting an attacker overwrite an existing tag with a malicious image.

```bash
REG=https://registry.example.com
curl -s $REG/v2/                                    # 200 with no auth = open
curl -s $REG/v2/_catalog                            # anonymous inventory

# Anonymous push: overwrite a tag that deployments pull
crane copy backdoored:latest $REG/<repo>:<tag>
```

## Exploitation notes

- A `200` on `/v2/` with no `Www-Authenticate` challenge signals an open registry.
- Overwriting a mutable tag (`latest`, an environment tag) is the highest-impact action: anything that pulls it runs your image.
- Where only pull is open, pivot to [Secrets in image layers](secrets-in-image-layers.md) across every repository.

## References

- [Docker registry HTTP API v2](https://distribution.github.io/distribution/spec/api/)
- [OCI distribution specification](https://github.com/opencontainers/distribution-spec)
