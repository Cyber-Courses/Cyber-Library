---
title: "Enumeration: discovering S3 buckets and their objects"
description: "Discovering S3 buckets and objects through naming guesses, DNS, and permission probing, then listing readable contents."
keywords:
  - S3 enumeration
  - bucket
  - ListBucket
  - object
  - discovery
---

# Enumeration

S3 buckets live in a single global namespace, so they can be found without any credential: a name either resolves or it does not. Enumeration is guessing names from the target's branding, confirming them over DNS or HTTP, and then probing what the current identity (or an anonymous caller) may list and read.

## Unauthenticated discovery

```bash
# a bucket name resolves in DNS if it exists
host target-backups.s3.amazonaws.com

# HTTP status distinguishes exists-but-denied (403) from absent (404)
curl -s -o /dev/null -w '%{http_code}\n' https://target-backups.s3.amazonaws.com

# anonymous list, if the bucket allows it
aws s3 ls s3://target-backups --no-sign-request
```

## Guessing names at scale

```bash
# permutation + wordlist scanning
s3scanner scan --bucket-file names.txt
cloud_enum -k target -k target-prod            # S3 + GCP + Azure in one pass
bucket-finder names.txt                        # classic read/list prober
# GrayhatWarfare: search already-indexed public buckets and their files by keyword (web UI/API)

# resolve a bucket's region from the endpoint redirect
dig +short <bucket>.s3.amazonaws.com
curl -sI https://<bucket>.s3.amazonaws.com | grep -i x-amz-bucket-region
```

## Authenticated listing

```bash
aws s3api list-buckets --query 'Buckets[].Name'
aws s3 ls s3://<bucket> --recursive
aws s3api list-objects-v2 --bucket <bucket> --query 'Contents[].Key'
```

## Recovering deleted and overwritten objects

A versioned bucket keeps every prior and deleted object; `ListBucket` hides them but `s3:ListBucketVersions` and `GetObjectVersion` pull them back, which often resurfaces secrets that were "removed":

```bash
aws s3api list-object-versions --bucket <bucket> \
  --query '{v:Versions[].{k:Key,id:VersionId},d:DeleteMarkers[].Key}'
# fetch a specific old or delete-marked version
aws s3api get-object --bucket <bucket> --key <key> --version-id <VersionId> out.bin
```

## Exploitation notes

- A `403` is a positive result: the bucket exists and is worth attacking by policy or ACL even when listing is denied.
- Object keys leak structure: dated backup prefixes, `terraform.tfstate`, and `.env` keys are high-value targets to request directly even without `ListBucket`.
- Region matters for the endpoint; a redirect in the response names the bucket's region.

## Tools

- **s3scanner**: bucket existence and permission scanning from a name list.
- **cloud_enum**: cross-provider public-resource discovery.
- **bucket-finder**: read and list probing across a name wordlist.
- **GrayhatWarfare**: searchable index of already-public buckets and files.
- **AWS CLI** (`--no-sign-request`, `list-object-versions`): anonymous list/get and version recovery.
- **Pacu** (`s3__bucket_finder`, `s3__download_bucket`): find and pull readable buckets across the account.

## References

- [s3scanner](https://github.com/sa7mon/S3Scanner)
- [cloud_enum (initstring)](https://github.com/initstring/cloud_enum)
- [GrayhatWarfare: public buckets](https://buckets.grayhatwarfare.com/)
- [HackTricks Cloud: S3 enumeration](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-services/aws-s3-athena-and-glacier-enum.html)
- [Hacking the Cloud: AWS](https://hackingthe.cloud/)
