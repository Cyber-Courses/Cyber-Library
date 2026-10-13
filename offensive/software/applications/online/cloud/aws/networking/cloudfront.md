---
title: "CloudFront: origin and cache behavior abuse"
order: 4
description: "Abusing CloudFront distributions, origins, and cache behavior to bypass controls or reach protected origins."
keywords:
  - CloudFront
  - distribution
  - origin
  - cache
  - CDN
---

# CloudFront

CloudFront sits in front of origins (S3 buckets, ALBs, custom hosts) and applies caching and request handling. Its configuration leaks the origin addresses the edge is meant to hide, and weak origin access or forwarded-header behavior lets you reach the origin directly or poison what other users receive. With `cloudfront:Get*`/`List*` you read every distribution's origins and behaviors.

## Reading distributions and origins

```bash
aws cloudfront list-distributions \
  --query "DistributionList.Items[].[Id,DomainName,Origins.Items[].DomainName]"
aws cloudfront get-distribution-config --id <dist-id>
```

## Reaching the origin directly

```bash
# origin domains revealed above are often reachable without the CDN in front,
# bypassing WAF/geo/signed-URL controls enforced only at the edge
curl -H 'Host: app.example.com' https://<origin-domain-from-config>/
```

## Exploitation notes

- Controls enforced only at the edge (WAF, signed URLs, geo restriction) are void if the origin accepts direct requests; the origin domain from the config is the bypass.
- An S3 origin without origin access control may be readable or claimable directly; a missing origin is a [dangling-DNS takeover](dangling-dns-takeover.md) candidate.
- Headers CloudFront forwards and keys the cache on determine whether a cache-poisoning or host-header payload reaches and sticks at the origin.

## Tools

- **AWS CLI** (`cloudfront get-distribution-config`): origin and behavior disclosure.
- **Burp Suite**: cache-poisoning and host-header testing against the edge.

## References

- [HackTricks Cloud: AWS CloudFront](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-services/aws-cloudfront-enum.html)
- [AWS: restricting access to an origin](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/private-content-restricting-access-to-origin.html)
- [can-i-take-over-xyz: CloudFront and subdomain takeover fingerprints](https://github.com/EdOverflow/can-i-take-over-xyz)
- [Hacking the Cloud: AWS offensive techniques](https://hackingthe.cloud/)
