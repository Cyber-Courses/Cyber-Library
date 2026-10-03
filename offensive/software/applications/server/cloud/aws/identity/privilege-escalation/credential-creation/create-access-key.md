---
title: "CreateAccessKey: mint keys for a more privileged user"
description: "iam:CreateAccessKey to mint a second set of long-term keys for a more privileged user."
keywords:
  - CreateAccessKey
  - access key
  - user
  - long-term credentials
  - IAM
---

# CreateAccessKey

`iam:CreateAccessKey` creates a long-term access key for a user. Run it against a more privileged user and you receive a working key pair for that identity, which you then use directly.

## Mint keys for a target

```bash
aws iam create-access-key --user-name <privileged-user>
# returns AccessKeyId + SecretAccessKey; configure a profile and use them
```

## Exploitation notes

- A user may hold at most two access keys; if the target already has two, this fails until one is deleted.
- The new key is a durable credential that survives your current session, so it doubles as persistence.
- No policy change is made on the target, so the escalation is quiet in policy-audit terms but does create a key-creation event.

## Tools

- **AWS CLI** (`iam create-access-key`).
- **Pacu** (`iam__backdoor_users_keys`): mints keys across users at scale.

## References

- [Rhino Security Labs: AWS privilege escalation (CreateAccessKey)](https://rhinosecuritylabs.com/aws/aws-privilege-escalation-methods-mitigation/)
- [BishopFox: iam-vulnerable](https://github.com/BishopFox/iam-vulnerable)
