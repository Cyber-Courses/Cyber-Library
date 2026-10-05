---
title: "Presigned URL: hand out time-limited access under your permissions"
description: "Minting presigned URLs for S3 objects and other resources to hand out time-limited access under your own permissions."
keywords:
  - presigned URL
  - S3
  - time-limited
  - signed request
  - access
---

# Presigned URL

A presigned URL embeds a signature made with your credentials, granting anyone who holds the URL the signed action for its lifetime. You mint presigned access to objects your principal can reach and share or stage it, moving data out without the recipient holding any AWS credentials.

## Presign an S3 object

```bash
aws s3 presign s3://<bucket>/<key> --expires-in 604800
# the returned URL downloads the object with no credentials, for 7 days
```

## Exploitation notes

- The URL carries your permissions, not the recipient's, so it is a clean exfiltration and sharing primitive.
- Presigned SageMaker and other service URLs work the same way: a signed, credential-free handle to a privileged action.
- The signature is valid until it expires or your credentials are revoked, so treat issued URLs as live secrets.

## Tools

- **AWS CLI** (`s3 presign`), **boto3** (`generate_presigned_url`) for arbitrary operations.

## References

- [AWS: using presigned URLs](https://docs.aws.amazon.com/AmazonS3/latest/userguide/using-presigned-url.html)
- [HackTricks Cloud: AWS S3](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-services/aws-s3-enum.html)
- [Pacu (Rhino Security Labs)](https://github.com/RhinoSecurityLabs/pacu)
- [Hacking the Cloud: AWS](https://hackingthe.cloud/)
