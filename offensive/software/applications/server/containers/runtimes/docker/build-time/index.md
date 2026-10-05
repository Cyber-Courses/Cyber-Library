---
title: "Build-time attacks: abusing the image build process"
description: "The container build is an execution environment with its own attack surface. Build arguments and mounted secrets leak into the final image or the build cache, and the BuildKit frontend and daemon have had flaws letting a crafted Dockerfile or build context read files outside the context or execute on the build host."
keywords:
  - docker build
  - buildkit
  - build-arg
  - build secret
  - build host
---

# Build-time attacks

Building an image runs commands, resolves a context, and caches intermediate results, all on a build host that frequently has more access than the eventual runtime: registry credentials, cloud roles for pushing, and the source repository. Two problems follow. Secrets supplied to the build leak into the image or cache where they can be recovered later. And the build tooling itself, BuildKit and the daemon, processes an attacker-influenced Dockerfile and context, so flaws there read host files or execute on the build host.

```bash
docker version | grep -i buildkit            # BuildKit frontend in use
env | grep -iE 'BUILD|REGISTRY|AWS|GITHUB_TOKEN'   # build-host credentials worth stealing
```

## Subtopics

- **[Build-arg secret leak](build-arg-secret-leak.md)**: secrets passed as build arguments persisting in the image.
- **[BuildKit RCE](buildkit-rce.md)**: crafted builds reading outside the context or executing on the build host.

## References

- [Docker: build secrets](https://docs.docker.com/build/building/secrets/)
- [BuildKit security advisories](https://github.com/moby/buildkit/security/advisories)
