---
title: "Weak passwords: guessing remote-desktop session and unattended passwords"
description: "Remote-desktop tools protect connections with a session or unattended password that is frequently weak: short auto-generated session passwords, user-chosen weak unattended passwords, and reused credentials. These are brute-forced or replayed against the ID, and historically weak rate-limiting on some products made online guessing of the short password space practical."
keywords:
  - weak password
  - session password
  - brute force
  - credential reuse
  - teamviewer
---

# Weak passwords

The password is what actually guards a remote-desktop connection, and it is often weak. Auto-generated session passwords are short (a handful of characters/digits) to be read aloud, shrinking the space; unattended passwords are user-chosen and frequently weak or reused; and both are subject to credential reuse from breaches and infostealer logs. An attacker brute-forces the password against a known ID or replays a leaked one. Historically, insufficient rate-limiting or lockout on some products made online guessing of the short session-password space practical at scale, which drove mass account-takeover campaigns; vendors have since added throttling, so current feasibility depends on the product and version.

```bash
# brute force / replay passwords against a known device ID
#   short auto-generated session passwords have a small space (version-dependent)
#   unattended passwords are guessed with common/reused lists
# replay leaked ID:password pairs from breaches/stealer logs for direct connects
```

## Exploitation notes

- Session passwords are short by design (meant to be spoken), so their space is small where rate-limiting is weak; unattended passwords are the more durable target and fall to common/reused lists.
- Credential reuse is the reliable modern path: ID-and-password pairs in breach and infostealer data connect directly without guessing, mirroring VPN/RDP credential stuffing.
- Vendor rate-limiting and lockout now blunt pure online brute force on current versions; the practicality depends on the product and its protections, so fingerprint the [version](../enumeration/version-detection.md).
- A guessed/replayed password plus the ID is interactive control; an unattended password is standing access, see [unattended access](unattended-access.md).

## References

- [TeamViewer security (password handling)](https://www.teamviewer.com/en/trust-center/security/)
- [HackTricks](https://book.hacktricks.xyz/)
