---
title: "Public access: reading and writing world-exposed buckets"
description: "Reading and writing objects in buckets left world-accessible by ACLs or by Block Public Access being disabled."
keywords:
  - S3
  - public access
  - Block Public Access
  - ACL
  - anonymous
---

# Public access

A bucket is public when **Block Public Access** is off and either its policy or an ACL grants the `AllUsers` or `AuthenticatedUsers` group. Read-public leaks the objects to anyone; write-public is worse, because an attacker who can `PutObject` over a bucket that serves a website, a script, or a software artifact controls what the victims fetch.

## Reading a public bucket

```bash
aws s3 ls s3://<bucket> --no-sign-request
aws s3 sync s3://<bucket> ./loot --no-sign-request
```

## Testing for public write

```bash
# AllUsers write: anyone can drop an object
echo probe > /tmp/probe.txt
aws s3 cp /tmp/probe.txt s3://<bucket>/probe.txt --no-sign-request
```

A successful anonymous `PutObject` means you can overwrite objects the bucket serves. Where the bucket backs a static site or a distributed script, replacing an object is a content-injection and code-execution path against its consumers.

## Checking the exposure controls

```bash
aws s3api get-public-access-block --bucket <bucket>
aws s3api get-bucket-acl --bucket <bucket>
```

## Exploitation notes

- `AuthenticatedUsers` means *any* AWS account, not the bucket's account; a bucket granting it is public to every attacker who holds any credential.
- Write-public buckets that hold JavaScript, firmware, or installer artifacts are the highest value: you move from data exposure to supply-style content injection on everyone who downloads them.
- Object-level ACLs can differ from the bucket default, so probe individual high-value keys even when the bucket listing is denied.

## Tools

- **AWS CLI** (`--no-sign-request`): anonymous read and write.
- **s3scanner**: flags read and write permission per bucket.

## References

- [HackTricks Cloud: S3 public access](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-services/aws-s3-athena-and-glacier-enum.html)
- [Rhino Security Labs: penetration testing AWS storage](https://rhinosecuritylabs.com/aws/penetration-testing-aws-storage/)
