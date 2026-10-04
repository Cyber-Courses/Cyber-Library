---
title: "Bucket policy and ACL: cross-account read, write, and takeover"
description: "Abusing over-broad bucket policies and object ACLs to read, write, or take over bucket contents across accounts."
keywords:
  - bucket policy
  - ACL
  - cross-account
  - PutObject
  - grant
---

# Bucket policy and ACL

Beyond the public/private switch, a bucket's **resource policy** and its **ACLs** grant specific principals specific actions, and both are routinely written too broadly: a policy that names a whole account as principal, or a `Principal: "*"` with a weak condition, hands access to attackers who satisfy the condition. If you can read the policy you learn who can reach the bucket; if you can *write* it (`s3:PutBucketPolicy`), you grant yourself whatever you lack.

## Reading the grants

```bash
aws s3api get-bucket-policy --bucket <bucket> --query Policy --output text | jq .
aws s3api get-bucket-acl --bucket <bucket>
aws s3api get-object-acl --bucket <bucket> --key <key>
```

## Rewriting a policy you can set

```bash
cat > p.json <<'EOF'
{"Version":"2012-10-17","Statement":[{"Effect":"Allow",
  "Principal":{"AWS":"arn:aws:iam::<your-acct>:root"},
  "Action":"s3:*","Resource":["arn:aws:s3:::<bucket>","arn:aws:s3:::<bucket>/*"]}]}
EOF
aws s3api put-bucket-policy --bucket <bucket> --policy file://p.json
```

## Cross-account grants to look for

- A statement whose `Principal.AWS` is another account root: any principal there with the matching S3 action reaches in.
- `s3:PutObject` granted broadly over a bucket consumed by a pipeline: write poisoned input and let the victim process it.
- Object ACLs granting `AllUsers`/`AuthenticatedUsers` even when the bucket policy looks tight.

## Exploitation notes

- Bucket ownership controls and the object-owner setting change whether an uploaded object's ACL is honored; where `BucketOwnerEnforced` is off, a cross-account writer can set ACLs on the objects it drops.
- `s3:PutBucketPolicy` on a bucket you do not own is itself a takeover primitive: rewrite the policy, then read or overwrite everything.
- Pair with [enumeration](enumeration.md) to find the buckets and with [public access](public-access.md) for the anonymous case.

## Tools

- **AWS CLI** (`s3api get-/put-bucket-policy`, `get-object-acl`): read and rewrite grants.
- **Pacu** (`s3__bucket_finder`): locate and assess buckets in a session.

## References

- [HackTricks Cloud: S3 bucket policy and ACL abuse](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-services/aws-s3-athena-and-glacier-enum.html)
- [AWS: bucket policy and ACL evaluation](https://docs.aws.amazon.com/AmazonS3/latest/userguide/access-policy-language-overview.html)
- [Hacking the Cloud: AWS](https://hackingthe.cloud/)
- [GrayhatWarfare: public buckets](https://buckets.grayhatwarfare.com/)
- [S3Scanner](https://github.com/sa7mon/S3Scanner)
- [cloud_enum](https://github.com/initstring/cloud_enum)
