---
title: "Dangling-DNS takeover: claiming an orphaned record's backing resource"
order: 5
description: "Claiming the backing resource of a dangling DNS record (S3, CloudFront, or ELB origin) to take over the subdomain."
keywords:
  - dangling DNS
  - subdomain takeover
  - CNAME
  - S3
  - CloudFront
---

# Dangling-DNS takeover

A dangling record points a name at an AWS resource that no longer exists: an S3 website bucket that was deleted, a released Elastic IP, a torn-down CloudFront distribution, or a removed ELB. Because the name still resolves to the provider, anyone who can recreate a resource with the same identifier captures every request to that subdomain, which yields a trusted-origin foothold for phishing, cookie theft, and certificate issuance.

## Finding dangling records

```bash
# records whose alias/CNAME targets an AWS resource
aws route53 list-resource-record-sets --hosted-zone-id <zone-id> \
  --query "ResourceRecordSets[?Type=='CNAME' || AliasTarget].[Name,Type,ResourceRecords[0].Value,AliasTarget.DNSName]"
# then resolve each target and flag NXDOMAIN / NoSuchBucket / unclaimed endpoints
```

## Claiming the backing resource

- **S3 website origin**: the target `<name>.s3-website-<region>.amazonaws.com` returning `NoSuchBucket` means you create a bucket with that exact name in that region and serve your content.
- **CloudFront**: a CNAME to a removed distribution's `*.cloudfront.net` can be claimed by registering the alternate domain name on a distribution you own.
- **Elastic IP / ELB**: an A record to a released EIP is captured by reallocating EIPs until you draw the same address; a dead ELB DNS name is reclaimed by creating a load balancer that is issued it.

## Exploitation notes

- A captured subdomain inherits the parent domain's trust: session cookies scoped to `.example.com`, SSO redirect allow-lists, and user expectation all transfer.
- Control of the name also passes ACME HTTP-01 and DNS-01 challenges, so you can mint a valid certificate for the subdomain.
- Cross-reference [S3 enumeration](../storage/s3/enumeration.md) for discovering the bucket-origin case and [Route53](route53.md) for the record inventory.

## Tools

- **nuclei** (`takeovers` templates) and **subjack** / **tko-subs**: detect claimable fingerprints at scale.
- **AWS CLI**: recreate the bucket, distribution alias, or EIP to claim the target.

## References

- [HackTricks Cloud: subdomain takeover](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-services/aws-route53-enum.html)
- [Can I take over XYZ (second-order takeover fingerprints)](https://github.com/EdOverflow/can-i-take-over-xyz)
- [Hacking the Cloud: AWS offensive techniques](https://hackingthe.cloud/)
- [fwd:cloudsec: cloud security conference talks](https://fwdcloudsec.org/)
