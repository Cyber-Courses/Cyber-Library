---
title: "STS tokens: capturing and replaying temporary session credentials"
description: "Capturing and replaying short-lived STS session tokens (AccessKeyId, SecretAccessKey, SessionToken) lifted from processes, logs, and environment variables."
keywords:
  - STS
  - session token
  - temporary credentials
  - SessionToken
  - replay
---

# STS tokens

Temporary STS credentials are a triple: an `ASIA`-prefixed access key, a secret, and a **session token**, with an expiry. They are what roles, federation, and the metadata service hand out, and they are reusable from anywhere until they expire, so lifting them from a process, a log line, or an environment variable is often enough to continue as the principal.

## Capturing and replaying

```bash
# in a compromised process environment
env | grep -E 'AWS_(ACCESS_KEY_ID|SECRET_ACCESS_KEY|SESSION_TOKEN)'

export AWS_ACCESS_KEY_ID=ASIA... \
       AWS_SECRET_ACCESS_KEY=... \
       AWS_SESSION_TOKEN=...
aws sts get-caller-identity           # confirm identity and that it is still valid
```

## Checking scope and lifetime

```bash
aws sts get-caller-identity           # the assumed-role ARN tells you what you are
# decode the token's expiry from the source that issued it (role max-session-duration)
```

## Exploitation notes

- `ASIA` keys without the session token are useless; always grab all three.
- Tokens from role chaining are capped at one hour; tokens from a direct assume can last up to the role's max session duration.
- Session credentials often appear in CI logs, crash dumps, and shell history, which are quieter sources than live memory.

## Tools

- **AWS CLI** (`sts get-caller-identity`): validate and attribute.
- **Pacu**: import session credentials and continue enumeration.

## References

- [AWS: temporary security credentials](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_temp.html)
- [HackTricks Cloud: STS](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-privilege-escalation/aws-sts-privesc.html)
