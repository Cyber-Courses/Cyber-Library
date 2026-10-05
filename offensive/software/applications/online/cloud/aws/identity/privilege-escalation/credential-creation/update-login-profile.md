---
title: "UpdateLoginProfile: reset a privileged user's console password"
description: "iam:UpdateLoginProfile to reset the console password of a more privileged user and take over their sign-in."
keywords:
  - UpdateLoginProfile
  - console password
  - reset
  - user
  - login profile
---

# UpdateLoginProfile

`iam:UpdateLoginProfile` resets the console password of a user that already has a login profile. Against a more privileged user, set a password you know and sign in as them.

## Reset the password

```bash
aws iam update-login-profile --user-name <privileged-user> \
  --password 'Newpass123!' --no-password-reset-required
```

## Exploitation notes

- This is disruptive if the user actively signs in, since their old password stops working, so it is a deliberate choice against accounts that are not in interactive use.
- If the target has no profile yet, use [CreateLoginProfile](create-login-profile.md) instead.
- Console MFA, where enforced, still gates sign-in even after a password reset.

## Tools

- **AWS CLI** (`iam update-login-profile`).
- **Pacu** (`iam__privesc_scan`).

## References

- [Rhino Security Labs: AWS privilege escalation (UpdateLoginProfile)](https://rhinosecuritylabs.com/aws/aws-privilege-escalation-methods-mitigation/)
- [BishopFox: iam-vulnerable](https://github.com/BishopFox/iam-vulnerable)
- [Pacu (Rhino Security Labs)](https://github.com/RhinoSecurityLabs/pacu)
- [Hacking the Cloud: AWS](https://hackingthe.cloud/)
