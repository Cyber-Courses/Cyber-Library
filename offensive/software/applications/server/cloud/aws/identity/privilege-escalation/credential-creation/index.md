---
title: "Credential creation"
description: "Minting new credentials for an IAM principal you can write to: additional access keys and new or reset console login profiles."
keywords:
  - CreateAccessKey
  - CreateLoginProfile
  - UpdateLoginProfile
  - access key
  - login profile
  - credentials
---

# Credential creation

If you can create or reset credentials on a more privileged principal, you become that principal without touching its policies. Two primitives: programmatic **access keys**, and console **login profiles**.

## Paths

- **[CreateAccessKey](create-access-key.md)**: mint a second set of long-term keys for a privileged user.
- **[CreateLoginProfile](create-login-profile.md)**: set a console password on a user that has none.
- **[UpdateLoginProfile](update-login-profile.md)**: reset the console password of a privileged user.

## References

- [Rhino Security Labs: AWS privilege escalation (credential paths)](https://rhinosecuritylabs.com/aws/aws-privilege-escalation-methods-mitigation/)
- [BishopFox: iam-vulnerable](https://github.com/BishopFox/iam-vulnerable)
