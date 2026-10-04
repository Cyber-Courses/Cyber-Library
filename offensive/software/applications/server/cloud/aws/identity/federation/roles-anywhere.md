---
title: "Roles Anywhere: role credentials from a trust anchor certificate"
description: "Abusing IAM Roles Anywhere trust anchors and client certificates to obtain role credentials from outside AWS."
keywords:
  - Roles Anywhere
  - trust anchor
  - certificate
  - X.509
  - role credentials
---

# Roles Anywhere

IAM Roles Anywhere lets workloads outside AWS obtain role credentials by presenting an X.509 client certificate issued under a configured **trust anchor** (a CA). If you can obtain or issue a certificate the trust anchor accepts, you mint role credentials from anywhere, no AWS user needed.

## Exchanging a certificate for credentials

```bash
aws_signing_helper credential-process \
  --certificate client.pem --private-key client.key \
  --trust-anchor-arn arn:aws:rolesanywhere:...:trust-anchor/<id> \
  --profile-arn arn:aws:rolesanywhere:...:profile/<id> \
  --role-arn arn:aws:iam::<acct>:role/<role>
```

## Exploitation notes

- The attack hinges on the CA: control of the trust-anchor CA (or a sub-CA it chains to) lets you issue certificates for any role the profile allows.
- A leaked client certificate and key are directly reusable until revoked or expired.
- The profile and role constrain which roles a certificate can assume, so enumerate profiles to see the reachable set.

## Tools

- **aws_signing_helper**: the AWS credential helper for Roles Anywhere.
- Standard PKI tooling (OpenSSL) to inspect or issue certificates.

## References

- [AWS: IAM Roles Anywhere](https://docs.aws.amazon.com/rolesanywhere/latest/userguide/introduction.html)
- [HackTricks Cloud: AWS Roles Anywhere](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-privilege-escalation/aws-sts-privesc.html)
- [Hacking the Cloud: AWS](https://hackingthe.cloud/)
- [Stratus Red Team: AWS techniques](https://stratus-red-team.cloud/attack-techniques/AWS/)
