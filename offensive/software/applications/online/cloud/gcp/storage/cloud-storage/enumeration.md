---
title: "Enumeration: discovering GCP buckets and readable objects"
order: 1
description: "Finding GCP buckets and readable objects through name guessing and listing, with tooling like GCPBucketBrute."
keywords:
  - bucket enumeration
  - GCPBucketBrute
  - public bucket
  - object listing
  - GCS
---

# Enumeration

GCS bucket names are global, so a name either exists or it does not, and existence is testable with no credential. Enumeration is guessing names from the target's branding, checking them, and then listing what an anonymous caller or your current identity may read.

## Unauthenticated discovery

```bash
# a bucket's JSON API endpoint distinguishes 200/403 (exists) from 404 (absent)
curl -s -o /dev/null -w '%{http_code}\n' \
  "https://storage.googleapis.com/storage/v1/b/target-backups"

# anonymous object list, if the bucket allows it
gcloud storage ls gs://target-backups --no-user-output-enabled 2>/dev/null
gsutil ls gs://target-backups
```

## Guessing names at scale

```bash
# GCPBucketBrute: permute a keyword, test existence and the ACL/IAM exposure,
# and flag buckets that allow privilege escalation (setIamPolicy by allUsers)
python3 gcpbucketbrute.py -k target -u          # -u = unauthenticated
cloud_enum -k target -k target-prod             # GCS + S3 + Azure in one pass
```

## Authenticated listing

```bash
gcloud storage buckets list --project <proj>
gcloud storage ls --recursive gs://<bucket>
gsutil iam get gs://<bucket>                    # read the bucket IAM policy
```

## Exploitation notes

- GCPBucketBrute flags buckets whose IAM lets `allUsers` call `storage.buckets.setIamPolicy`, which is a direct takeover of the bucket.
- A `403` is a positive hit: the bucket exists and is worth attacking by IAM or ACL even when listing is denied.
- Object names leak structure; request `terraform.tfstate`, `.env`, and dated backup objects directly even without list permission.

## Tools

- **GCPBucketBrute**: existence, ACL/IAM exposure, and privesc detection from a keyword.
- **cloud_enum**: cross-provider public-bucket discovery.
- **gsutil / gcloud storage**: anonymous and authenticated listing.

## References

- [GCPBucketBrute (Rhino Security Labs)](https://github.com/RhinoSecurityLabs/GCPBucketBrute)
- [cloud_enum (initstring)](https://github.com/initstring/cloud_enum)
- [HackTricks Cloud: GCP buckets enumeration](https://cloud.hacktricks.wiki/en/pentesting-cloud/gcp-security/index.html)
- [Hacking the Cloud: GCP](https://hackingthe.cloud/)
