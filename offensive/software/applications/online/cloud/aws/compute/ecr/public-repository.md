---
title: "Public repository: mining secrets from exposed ECR images"
description: "Pulling images from public or loosely-scoped ECR repositories to mine secrets and understand internal services."
keywords:
  - ECR
  - public repository
  - image pull
  - secrets
  - registry
---

# Public repository

ECR repositories can be made public through the ECR Public gallery, or left pullable by an over-broad repository policy. Either way, anyone who can pull the image gets its full layer history, which frequently holds hardcoded credentials, API tokens, internal hostnames, and source code.

## Pulling and inspecting

```bash
# Public gallery image
docker pull public.ecr.aws/<alias>/<repo>:<tag>

# Inspect layers and history for secrets
docker history --no-trunc <image>
dive <image>            # interactive layer explorer
```

Scan the unpacked layers with a secret scanner rather than reading by hand:

```bash
docker save <image> -o img.tar && mkdir x && tar -xf img.tar -C x
trufflehog filesystem x/
```

## Exploitation notes

- Secrets in an early layer persist even if a later layer deletes the file, so always scan the full history, not the final filesystem.
- Internal image names and tags leak the service inventory and version, which seeds targeting of the running workloads.
- A repository left pullable cross-account is the [repository policy](repository-policy.md) case; a truly public one needs no credentials at all.

## Tools

- **docker** / **crane**: pull and export images.
- **dive**: inspect layers interactively.
- **TruffleHog** / **gitleaks**: scan extracted layers for secrets.

## References

- [HackTricks Cloud: ECR](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-services/aws-ecr-enum.html)
- [TruffleHog (Truffle Security)](https://github.com/trufflesecurity/trufflehog)
- [Hacking the Cloud: AWS](https://hackingthe.cloud/)
- [Stratus Red Team: AWS techniques](https://stratus-red-team.cloud/attack-techniques/AWS/)
- [CloudFox (Bishop Fox)](https://github.com/BishopFox/cloudfox)
