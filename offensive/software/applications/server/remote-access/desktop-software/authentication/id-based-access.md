---
title: "ID-based access: enumerating and targeting device IDs"
description: "Remote-desktop tools address each machine by a numeric device ID, and connecting needs the ID plus a password. Because IDs are allocated from a predictable space, an attacker enumerates valid IDs and pairs them with password guessing or stolen passwords. Historically, weak ID-to-session binding and predictable IDs made brute-forcing the ID-plus-password combination practical."
keywords:
  - device id
  - teamviewer id
  - anydesk id
  - id enumeration
  - brute force
---

# ID-based access

Products like TeamViewer and AnyDesk identify each installation by a numeric ID, and a remote connection is established by supplying that ID and the corresponding password. The ID is the addressing, and its properties shape the attack: IDs are drawn from a bounded, often sequential-ish space, so valid installations can be enumerated, and an attacker pairs a discovered ID with password guessing or a stolen/leaked password to connect. The strength of the binding between ID and session password matters, where it was weak or the ID space predictable, brute-forcing the ID-and-password combination at scale was practical, and mass campaigns have abused reused passwords across enumerated IDs.

```bash
# IDs are numeric and enumerable; connections pair ID + password
# an attacker sweeps candidate IDs and attempts known/weak passwords against each
#   (automated against the client/relay connection flow)
# combine with leaked ID:password pairs (from breaches/stealer logs) for direct connects
```

## Exploitation notes

- The ID is addressing, not a secret; security rests on the password, so the real attack is the ID enumeration plus a password source (guessing, reuse, or leaked pairs), see [weak passwords](weak-passwords.md).
- Infostealer logs and breach data frequently contain ID-and-password pairs for these tools, making many "connects" direct replays rather than brute force.
- Historically weak ID-to-session binding and rate limiting let attackers brute-force the combination; vendors have tightened this, so current practicality depends on the product version and its protections.
- A connected session is interactive control of the remote machine as its logged-in user; unattended-access configs make this reachable at any time, see [unattended access](unattended-access.md).

## References

- [TeamViewer ID and connection model](https://www.teamviewer.com/en/)
- [HackTricks](https://book.hacktricks.xyz/)
