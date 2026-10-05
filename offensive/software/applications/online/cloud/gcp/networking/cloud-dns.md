---
title: "Cloud DNS: hijacking dangling records for subdomain takeover"
description: "Hijacking dangling Cloud DNS records that point to released GCP resources for subdomain takeover."
keywords:
  - Cloud DNS
  - dangling DNS
  - subdomain takeover
  - DNS record
  - hijack
  - GCP
---

# Cloud DNS

A dangling Cloud DNS record points a name at a GCP resource that no longer exists: a released global or regional static IP, a torn-down GCS website bucket, an App Engine or Cloud Run custom-domain mapping that was removed, or a deleted load-balancer address. Because the name still resolves toward GCP, whoever recreates a resource that draws the same address or identifier captures every request to that subdomain, which yields a trusted-origin foothold for phishing, cookie theft, and certificate issuance.

## Finding dangling records

```bash
# every managed zone and its records
gcloud dns managed-zones list
gcloud dns record-sets list --zone <zone> \
  --format="table(name,type,rrdatas.list())"
# resolve each A / CNAME target and flag released IPs, NXDOMAIN, or NoSuchBucket
```

## Claiming the backing resource

- **Released static IP**: an A record to a released global/regional address is captured by allocating addresses (`gcloud compute addresses create`) until GCP reissues the same one.
- **GCS website**: a CNAME to `c.storage.googleapis.com` for a deleted bucket is reclaimed by creating a bucket with the exact name.
- **App Engine / Cloud Run / GCLB**: a custom-domain mapping to a removed service is reclaimed by mapping the same hostname to a service you control in your own project.

## Exploitation notes

- A captured subdomain inherits the parent domain's trust: cookies scoped to the parent, SSO redirect allow-lists, and user expectation all transfer.
- Control of the name passes ACME HTTP-01 and DNS-01 challenges, so you mint a valid certificate for the subdomain.
- The `rrdatas` of an A record that no longer answers, or a CNAME to an unclaimed GCP endpoint, is the fingerprint to hunt for at scale.

## Tools

- **gcloud** (`dns record-sets list`): record inventory.
- **nuclei** (`takeovers` templates), **subjack** / **tko-subs**: detect claimable fingerprints at scale.
- **can-i-take-over-xyz**: the fingerprint reference for which providers are claimable.

## References

- [can-i-take-over-xyz (subdomain takeover fingerprints)](https://github.com/EdOverflow/can-i-take-over-xyz)
- [HackTricks Cloud: GCP DNS](https://cloud.hacktricks.wiki/en/pentesting-cloud/gcp-security/index.html)
- [Hacking the Cloud: GCP](https://hackingthe.cloud/)
