---
title: "CreateLoginProfile: set a console password on a user with none"
description: "iam:CreateLoginProfile to set a console password on a user that has none, enabling interactive sign-in as that principal."
keywords:
  - CreateLoginProfile
  - console password
  - login profile
  - user
  - sign-in
---

# CreateLoginProfile

A user with programmatic keys but no console login profile cannot sign in to the web console. `iam:CreateLoginProfile` sets one, so against a privileged user that lacks a profile you create a password and sign in interactively as them.

## Create a profile

```bash
aws iam create-login-profile --user-name <privileged-user> \
  --password 'Newpass123!' --no-password-reset-required
# then sign in at the account console URL as that user
```

## Exploitation notes

- Fails if the user already has a login profile; use [UpdateLoginProfile](update-login-profile.md) to reset an existing one instead.
- `--no-password-reset-required` avoids the forced change-on-first-login prompt.
- Console access can reach actions and views that scripting around the CLI makes awkward, and carries the user's full permissions.

## Tools

- **AWS CLI** (`iam create-login-profile`).
- **Pacu** (`iam__privesc_scan`).

## References

- [Rhino Security Labs: AWS privilege escalation (CreateLoginProfile)](https://rhinosecuritylabs.com/aws/aws-privilege-escalation-methods-mitigation/)
- [BishopFox: iam-vulnerable](https://github.com/BishopFox/iam-vulnerable)
