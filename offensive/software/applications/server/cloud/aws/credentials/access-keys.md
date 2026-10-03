---
title: "Access keys: long-term AKIA pairs in files, env, and source"
description: "Finding and using long-term AWS access key pairs left in files, environment variables, CI configuration, and source code including git history and Docker layers."
keywords:
  - access keys
  - AKIA
  - credentials file
  - environment variables
  - git history
---

# Access keys

A long-term access key is an `AKIA`-prefixed ID and its secret. Unlike role credentials they do not expire, so a key found once keeps working until it is rotated or disabled. They leak constantly into files, environment variables, CI configuration, and committed source, which makes grepping for them one of the highest-yield moves on any foothold.

## Where they sit on a host

```bash
cat ~/.aws/credentials ~/.aws/config 2>/dev/null
env | grep -i AWS_                       # AWS_ACCESS_KEY_ID / AWS_SECRET_ACCESS_KEY
cat /proc/*/environ 2>/dev/null | tr '\0' '\n' | grep AWS_
```

## In source and history

```bash
# live tree and full git history
trufflehog filesystem . ; trufflehog git file://. 
gitleaks detect --source . -v
# Docker images bake keys into layers
trufflehog docker --image <image:tag>
```

## Using and triaging a key

```bash
export AWS_ACCESS_KEY_ID=AKIA... AWS_SECRET_ACCESS_KEY=...
aws sts get-caller-identity          # whose key is it, which account
```

## Exploitation notes

- An `ASIA` prefix is a temporary key and also needs `AWS_SESSION_TOKEN`; see [STS tokens](sts-tokens.md).
- Keys in git history survive a later deletion commit, so always scan the full history, not just the checkout.
- Once validated, resolve the principal's rights with [identity enumeration](../identity/enumeration.md) before acting.

## Tools

- **TruffleHog** / **Gitleaks**: secret scanning across filesystem, git, and images.
- **AWS CLI** (`sts get-caller-identity`): validate and attribute a key.

## References

- [TruffleHog (Truffle Security)](https://github.com/trufflesecurity/trufflehog)
- [HackTricks Cloud: AWS access keys](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/index.html)
