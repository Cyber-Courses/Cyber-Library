---
title: "SSRF to IMDS: stealing a managed-identity token through a web app"
description: "Reaching the Azure IMDS endpoint through a server-side request forgery in an application to steal the host's managed-identity token."
keywords:
  - SSRF
  - IMDS
  - managed identity
  - token theft
  - metadata endpoint
---

# SSRF to IMDS

When an application on an Azure resource can be coerced into fetching an attacker-chosen URL, point it at the metadata token endpoint and the response is the host's managed-identity token. This is the bridge from a web vulnerability to full Azure credentials, which is why [SSRF](../../../../../server/web/code/injection/request-forgery/index.md) is treated as a cloud-credential technique.

## The core request

```
http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https://management.azure.com/
```

The catch unique to Azure: IMDS requires the header **`Metadata: true`** and rejects requests carrying an `X-Forwarded-For`. So the SSRF sink must let you set a header, or be a sink that forwards your headers. Where the app sets its own headers you cannot, the endpoint refuses, which shapes which sinks are usable.

```
# sinks that let you control headers or the raw request are the usable ones:
#   a proxy/webhook that forwards caller headers, a PDF/SSRF gadget with header injection,
#   or a gopher:// sink that writes the full HTTP request including Metadata: true
```

## Bypasses when the literal IP is filtered

```
http://169.254.169.254/   ->  http://[::ffff:169.254.169.254]/   (IPv6-mapped)
                              http://2852039166/                   (decimal)
# DNS rebinding and open-redirect chains where the fetcher follows 3xx
```

## Exploitation notes

- The web SSRF mechanics (sinks, filters, redirect and rebinding bypasses) live in [request forgery](../../../../../server/web/code/injection/request-forgery/index.md); this page is only the Azure payload and the `Metadata: true` constraint.
- Mint the `management.azure.com` token first to map the identity's RBAC, then re-request for `graph.microsoft.com` or `vault.azure.net` as needed.
- App Service SSRF cannot reach 169.254.169.254 (no IMDS there); target the `IDENTITY_ENDPOINT` env value instead, which an SSRF that leaks environment or hits the local port can reach.

## Tools

- **Burp Suite** or a custom client to drive the SSRF sink with the header.
- **az rest** / curl to replay the stolen token.

## References

- [HackTricks Cloud: SSRF to Azure IMDS](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
- [Microsoft: IMDS security (Metadata header)](https://learn.microsoft.com/azure/virtual-machines/instance-metadata-service)
- [ROADtools (Dirk-jan Mollema)](https://github.com/dirkjanm/ROADtools)
