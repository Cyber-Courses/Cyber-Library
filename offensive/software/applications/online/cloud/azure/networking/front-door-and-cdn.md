---
title: "Front Door and CDN: origin exposure and routing abuse"
description: "Abusing Azure Front Door and CDN: origin exposure, host-header and routing abuse, and dangling endpoints."
keywords:
  - Front Door
  - CDN
  - origin
  - host header
  - routing
---

# Front Door and CDN

Azure Front Door and CDN sit in front of application origins. The recurring weaknesses are an **origin** that accepts traffic directly (bypassing the WAF and routing rules at the edge), **host-header** handling that lets a request be routed or cached for a domain it should not, and **dangling endpoints** left when a profile is deleted but its DNS record is not.

## Enumerating profiles and origins

```bash
# Front Door profiles, their endpoints and origin groups
az afd profile list -o table
az afd origin-group list --profile-name <p> -g <rg> -o table
az afd origin list --profile-name <p> -g <rg> --origin-group-name <og> \
  --query "[].{Host:hostName,Origin:name}" -o table
# classic CDN endpoints
az cdn endpoint list --profile-name <p> -g <rg> --query "[].{Name:name,Origin:origins[0].hostName}" -o table
```

## Exploitation notes

- If the origin host (an App Service, storage, or VM) is reachable directly, requests sent to it skip the Front Door WAF and any path or header routing, so the edge controls are moot; find the origin and hit it directly.
- Host-header or caching rules that key on an attacker-controllable value can poison the cache or route to the wrong backend.
- A deleted Front Door or CDN endpoint whose `*.azureedge.net` / `*.azurefd.net` name still has a CNAME is a [DNS takeover](dns-takeover.md).

## Tools

- **az afd** / **az cdn**: profile, endpoint, and origin enumeration.
- **Burp Suite**: host-header and cache-key testing against the edge and origin.

## References

- [HackTricks Cloud: Azure services](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
- [Microsoft: Azure Front Door](https://learn.microsoft.com/azure/frontdoor/front-door-overview)
- [Can I take over XYZ (edge endpoint fingerprints)](https://github.com/EdOverflow/can-i-take-over-xyz)
