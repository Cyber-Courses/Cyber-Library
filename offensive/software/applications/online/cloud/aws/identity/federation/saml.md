---
title: "SAML: forging assertions for AssumeRoleWithSAML"
order: 1
description: "Forging or replaying SAML assertions trusted by an IAM SAML provider to assume federated roles via AssumeRoleWithSAML."
keywords:
  - SAML
  - assertion
  - federation
  - IAM provider
  - AssumeRoleWithSAML
---

# SAML

An IAM SAML identity provider trusts assertions signed by an external IdP's certificate. `sts:AssumeRoleWithSAML` exchanges a valid assertion for role credentials. If you can sign assertions (by compromising the IdP signing key) or replay a captured one, you mint credentials for any role whose trust and assertion attributes you satisfy.

## Assume with an assertion

```bash
aws sts assume-role-with-saml \
  --role-arn arn:aws:iam::<acct>:role/<role> \
  --principal-arn arn:aws:iam::<acct>:saml-provider/<provider> \
  --saml-assertion "$(base64 -w0 assertion.xml)"
```

## Exploitation notes

- The assertion's `Role` attribute lists role/provider ARN pairs; the role's trust must name the SAML provider as `Principal.Federated`.
- Control of the IdP's signing key is golden: it forges assertions for any mapped role, the SAML analogue of a golden ticket.
- Captured assertions are replayable until they expire, so intercepting one in transit is enough for a short window.

## Tools

- **AWS CLI** (`sts assume-role-with-saml`).
- **python3-saml** / custom tooling to craft or re-sign assertions.

## References

- [AWS: SAML 2.0 federation](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_providers_saml.html)
- [HackTricks Cloud: AWS federation](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-privilege-escalation/aws-sts-privesc.html)
- [Hacking the Cloud: AWS](https://hackingthe.cloud/)
- [Stratus Red Team: AWS techniques](https://stratus-red-team.cloud/attack-techniques/AWS/)
