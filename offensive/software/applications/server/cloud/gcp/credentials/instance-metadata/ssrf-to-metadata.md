---
title: "SSRF to metadata: stealing the service-account token through a hosted app"
description: "Reaching metadata.google.internal through a server-side request forgery in a hosted app to steal the instance's service-account token."
keywords:
  - SSRF
  - metadata server
  - service account token
  - Metadata-Flavor
  - request forgery
  - GCE
---

# SSRF to metadata

When an app running on Compute Engine, GKE, or Cloud Run can be coerced into fetching an attacker-chosen URL, point it at the metadata server and the response carries the instance's service-account token. This is the bridge from a web vulnerability to full cloud credentials, which is why [SSRF](../../../../web/code/injection/request-forgery/index.md) is treated here as a cloud-credential technique.

## The core request

```
http://metadata.google.internal/computeMetadata/v1/instance/service-accounts/default/token
http://169.254.169.254/computeMetadata/v1/instance/service-accounts/default/token
```

GCP metadata requires the header `Metadata-Flavor: Google` (older deployments also honored `X-Google-Metadata-Request: True`). An SSRF that cannot set headers fails against a correctly behaving server, so the usable cases are sinks that forward headers, that follow a redirect to a URL you control which then sets the header, or legacy endpoints.

## Bypasses

```
# alternate host spellings for IP/host filters
http://metadata.google.internal/  ->  http://169.254.169.254/
                                       http://[::ffff:169.254.169.254]/
                                       http://2852039166/            (decimal)
# recursive dump once you can reach it
.../computeMetadata/v1/?recursive=true&alt=json
# redirect chain: SSRF -> attacker redirector that responds with the header -> metadata
```

## Exploitation notes

- The web SSRF mechanics (sinks, filters, header injection, DNS rebinding) live in [request forgery](../../../../web/code/injection/request-forgery/index.md); this page is only the GCP payload.
- GKE pods often reach the node metadata server unless Workload Identity and metadata concealment are enforced; a pod SSRF then yields the node service account.
- Once the token is out, the attached account's scopes and roles decide reach; pivot to [service account keys](../service-account-keys.md) for persistence.

## Tools

- **Burp Suite** / custom scripts to drive the SSRF sink.
- **gcloud** / **curl** once the token is exported.

## References

- [Hacking the Cloud: GCP metadata SSRF](https://hackingthe.cloud/)
- [HackTricks Cloud: GCP SSRF to metadata](https://cloud.hacktricks.wiki/en/pentesting-cloud/gcp-security/index.html)
- [Google: metadata security](https://cloud.google.com/compute/docs/metadata/overview)
