---
title: "DNS takeover: claiming dangling Azure DNS records"
description: "Claiming dangling Azure DNS records left by deleted App Service, Traffic Manager, CDN, or public-IP resources."
keywords:
  - DNS takeover
  - dangling DNS
  - App Service
  - Traffic Manager
  - subdomain takeover
---

# DNS takeover

A dangling record points a name at an Azure resource that no longer exists: a deleted App Service (`*.azurewebsites.net`), a removed Traffic Manager profile (`*.trafficmanager.net`), a torn-down CDN or Front Door endpoint (`*.azureedge.net`), a deleted storage static site, or a released public IP. Because the name still resolves to the Azure namespace, whoever recreates a resource with the same identifier captures every request to that subdomain, a trusted-origin foothold for phishing, cookie theft, and certificate issuance.

## Finding dangling records

```bash
# records whose target is an Azure namespace, then resolve and flag NXDOMAIN / unclaimed
az network dns zone list -o table
az network dns record-set list -g <rg> -z <zone> \
  --query "[?cnameRecord].{Name:name,Target:cnameRecord.cname}" -o table
```

## Claiming the backing resource

- **App Service**: a CNAME to `victim.azurewebsites.net` returning NXDOMAIN means you create a Web App named `victim` in the same public region and serve content.
- **Traffic Manager / CDN / Front Door**: recreate a profile or endpoint with the same label (`victim.trafficmanager.net`, `victim.azureedge.net`) on a resource you own.
- **Storage static site / public IP**: recreate the storage account name, or reallocate public IPs until you draw the referenced address.

## Exploitation notes

- A captured subdomain inherits the parent domain's trust: cookies scoped to `.example.com`, SSO redirect allow-lists, and user expectation transfer.
- Control of the name passes ACME HTTP-01 and DNS-01 challenges, so you can mint a valid certificate for the subdomain.
- Microsoft's own guidance notes thousands of subdomains are vulnerable across tenants at any time because deletions rarely clean up the DNS record.

## Tools

- **subdomain-takeover fingerprints**: [can-i-take-over-xyz](https://github.com/EdOverflow/can-i-take-over-xyz) lists the Azure services and their claimable signatures.
- **nuclei** (`takeovers` templates), **subjack**, **tko-subs**: detect claimable records at scale.
- **az cli**: recreate the App Service, Traffic Manager profile, or storage account to claim it.

## References

- [HackTricks Cloud: Azure DNS takeover](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
- [Can I take over XYZ (takeover fingerprints)](https://github.com/EdOverflow/can-i-take-over-xyz)
- [Microsoft: prevent dangling DNS entries](https://learn.microsoft.com/azure/security/fundamentals/subdomain-takeover)
