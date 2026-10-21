---
title: "Weak passwords: guessing remote-desktop session and unattended passwords"
description: "Password-based remote access can be weakened by short, reused, or disclosed credentials, but password formats and online-guessing protections differ by product and release. Assess the configured authentication method, password policy, rate limiting, and additional access controls before claiming brute-force feasibility."
keywords:
  - weak password
  - session password
  - brute force
  - credential reuse
  - teamviewer
---

# Weak passwords

Some remote-desktop products support temporary session passwords, configured unattended-access passwords, account-based access, or passwordless device assignment. A weak, reused, or disclosed password can permit unauthorized access when it is the applicable authentication factor, but its exposure and guessability depend on the exact configuration. TeamViewer Classic, for example, supports random passwords of 6, 8, or 10 characters and applies increasing delays to failed connection attempts. CVE-2018-16550 documented a historical brute-force concern and must not be generalized to every TeamViewer or remote-desktop release.

```bash
# authorized assessment checklist
# - identify the active access method and password policy
# - verify rate limiting, lockout, approval, allowlist, and MFA controls
# - test only within the agreed attempt limits
```

## Exploitation notes

- A valid password and device ID may still be constrained by interactive approval, allowlists, multifactor authentication, account assignment, and permission profiles.
- Evaluate temporary and unattended credentials separately: they can have different lifetimes, policies, and secondary controls.
- Do not infer online-guessing feasibility from password length alone. Measure product-specific throttling or lockout only during an authorized test.
- Unattended access can provide recurring entry when valid reusable authentication material is obtained; see [unattended access](unattended-access.md).

## References

- [TeamViewer remote control via random password](https://www.teamviewer.com/en/global/support/knowledge-base/teamviewer-classic/remote-control/connection-methods/remote-control-via-random-password/)
- [TeamViewer brute-force protection](https://www.teamviewer.com/en-in/global/support/knowledge-base/teamviewer-classic/security/security-features/brute-force-protection/)
- [TeamViewer statement on CVE-2018-16550](https://community.teamviewer.com/English/discussion/43215/statement-on-recent-brute-force-research-cve-2018-16550)
- [AnyDesk unattended access](https://support.anydesk.com/docs/unattended-access)
