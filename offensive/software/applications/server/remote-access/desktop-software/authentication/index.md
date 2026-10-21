---
title: "Authentication: attacking remote-desktop software access control"
description: "Third-party remote-desktop tools use a device ID for addressing and product-specific controls for authentication and authorization. Review passwords, account assignment, user approval, allowlists, multifactor authentication, and unattended-access policy without treating the ID itself as a credential."
keywords:
  - id-based access
  - unattended access
  - weak password
  - teamviewer
  - anydesk
---

# Authentication

These products use a device identifier to address a connection and then apply product-specific authentication and authorization controls. Depending on the deployment, those controls may include a temporary or unattended password, account assignment, user approval, allowlists, multifactor authentication, and managed-device identity. The ID is targeting information rather than proof of access. Review the complete policy, including whether unattended access is enabled and whether any reusable authentication material is weak or exposed. A successful connection operates within the permissions granted by the product and the current operating-system session.

## Subtopics

- **[ID-based access](id-based-access.md)**: separating device addressing from authentication.
- **[Unattended access](unattended-access.md)**: reviewing always-on access controls and reusable authentication material.
- **[Weak passwords](weak-passwords.md)**: guessing session and unattended passwords.

## References

- [TeamViewer unattended access](https://www.teamviewer.com/en/)
- [HackTricks](https://book.hacktricks.xyz/)
