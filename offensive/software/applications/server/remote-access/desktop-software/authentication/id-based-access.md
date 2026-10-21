---
title: "ID-based access: identifiers are addressing, not authentication"
description: "Remote-desktop products use a device ID to address a connection, but the ID is not an authentication secret. Access still depends on the product's password, account, approval, certificate, allowlist, and policy controls; enumeration feasibility must be established for a specific product and version."
keywords:
  - device id
  - teamviewer id
  - anydesk id
  - id enumeration
  - brute force
---

# ID-based access

Products such as TeamViewer and AnyDesk use a device ID to address a connection. The ID is not intended to be a secret, but knowing it does not itself grant access: the product can additionally require a session or unattended password, account authorization, user approval, allowlist rules, multifactor controls, and device identity checks. AnyDesk, for example, links an ID to a client-certificate fingerprint. Claims that IDs can be enumerated or brute-forced must therefore be supported for the exact product, service behavior, and version rather than inferred from the identifier being numeric.

```bash
# collect an ID from authorized inventory, support records, or endpoint configuration
# then assess the actual access policy: password, approval, account assignment, and allowlist
```

## Exploitation notes

- Treat an exposed ID as targeting information, not a credential. Review every authentication and authorization control protecting the connection, see [weak passwords](weak-passwords.md).
- Credential or configuration material recovered from a host may include an ID, but whether it includes a reusable password depends on the product, version, and storage design.
- Do not claim large-scale ID enumeration without a reproducible product-specific method and evidence that the relay permits it.
- A connected session is interactive control of the remote machine as its logged-in user; unattended-access configs make this reachable at any time, see [unattended access](unattended-access.md).

## References

- [AnyDesk client fingerprint](https://support.anydesk.com/fingerprint)
