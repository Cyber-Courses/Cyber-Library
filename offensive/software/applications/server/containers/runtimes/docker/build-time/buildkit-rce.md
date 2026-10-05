---
title: "BuildKit RCE: escaping the build to read the host or execute on it"
description: "BuildKit resolves the build context, frontends, and cache mounts from an attacker-influenced Dockerfile. Flaws in that handling have let a crafted build read files outside the build context through cache or mount path traversal, follow symlinks into the host filesystem, and in some cases execute on the build host, compromising a CI runner or build server."
keywords:
  - buildkit
  - build context
  - cache mount
  - path traversal
  - build host
---

# BuildKit RCE

BuildKit is the modern Docker build backend. It interprets a Dockerfile (or an alternative frontend), fetches the build context, and manages cache and mount directives, all driven by input the attacker controls when they can submit a Dockerfile or build context, as in a CI system that builds user repositories. Several classes of flaw follow: cache and mount path handling that escapes the intended context, symlink following that reaches host files, and frontend parsing issues. The impact ranges from reading files outside the build context to executing on the build host, which in CI is often a credential-rich target.

Identify the build path and whether it is attacker-reachable:

```bash
docker buildx version 2>/dev/null; docker version | grep -i buildkit
# in CI: can an attacker submit the Dockerfile and context that gets built?
```

## Representative routes

```dockerfile
# 1. Cache or bind mount directives pointed outside the context, where a
#    traversal or symlink flaw lets the build read host files into a layer.
RUN --mount=type=cache,target=/c \
    cp /c/../../../../etc/shadow /out 2>/dev/null || true

# 2. A symlink in the build context that BuildKit follows during COPY,
#    resolving to a host path outside the context root.
#    (ship a symlink "ctx-link" -> / in the context, then COPY through it)
COPY ctx-link/etc/ /host-etc/
```

```bash
# 3. Frontend/daemon flaws: a crafted frontend reference or LLB that the
#    builder executes with builder privileges on the host. Version-specific;
#    fingerprint BuildKit and match to a known advisory.
```

The common mechanism is that a directive meant to be scoped to the build context is made to resolve a path on the build host, so content crosses from the host into the build (a read) or the builder performs an attacker-directed action (execution).

## Exploitation notes

- The realistic target is a shared build service or CI runner that builds untrusted Dockerfiles; there the build host holds registry credentials, cloud roles, and other tenants' sources.
- These are version-specific; fingerprint BuildKit and match against its advisories rather than assuming a generic primitive. Patched versions constrain mount and symlink resolution to the context.
- A read primitive that pulls host files into an output layer is often enough: steal the runner's credentials, then pivot with those rather than needing full execution.

## References

- [BuildKit security advisories](https://github.com/moby/buildkit/security/advisories)
- [Docker: build context and mounts](https://docs.docker.com/build/building/context/)
