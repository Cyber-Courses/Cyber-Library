---
title: "Credential brokers: services that mint credentials for other principals"
description: "Abusing services that broker credentials to other principals, turning a narrow permission into fresh role or user credentials."
keywords:
  - credential broker
  - role credentials
  - token
  - STS
  - privilege escalation
---

# Credential brokers

Several AWS actions exist to hand out credentials, and holding one of them turns a narrow permission into a fresh set of keys for another principal. These are the credential-minting counterparts to the [privilege-escalation](../identity/privilege-escalation/index.md) paths: where those change what a principal can do, these produce usable credentials directly.

## The brokering actions

```bash
# mint a second long-term key for a user you can write to
aws iam create-access-key --user-name <victim>

# a role's credentials via assume, where the trust allows you
aws sts assume-role --role-arn <arn> --role-session-name s

# EC2 Instance Connect pushes a key you then use to reach the box and its role
aws ec2-instance-connect send-ssh-public-key \
  --instance-id <id> --instance-os-user ec2-user --ssh-public-key file://key.pub
```

## Exploitation notes

- `iam:CreateAccessKey` on another user is the quietest broker: the new key is valid immediately and survives the victim's session ending, covered under [credential creation](../identity/privilege-escalation/credential-creation/index.md).
- `sts:AssumeRole` and the web-identity and SAML variants are brokers scoped by trust policy; see [role assumption](../identity/role-assumption/index.md) and [federation](../identity/federation/index.md).
- Service-linked brokers (Cognito identity pools handing unauthenticated role credentials) live under their service in [identity](../identity/cognito/index.md).

## Tools

- **AWS CLI** (`iam create-access-key`, `sts assume-role`, `ec2-instance-connect`).
- **Pacu**: chains brokering actions discovered by the privesc scan.

## References

- [Rhino Security Labs: AWS privilege escalation methods](https://rhinosecuritylabs.com/aws/aws-privilege-escalation-methods-mitigation/)
- [HackTricks Cloud: AWS credential abuse](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/index.html)
