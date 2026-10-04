---
title: "Guest access: B2B guests inside the tenant"
description: "Abusing B2B guest accounts: low-friction invitation, guest enumeration of the directory, and guest-to-member escalation paths."
keywords:
  - guest access
  - B2B
  - invitation
  - guest enumeration
  - external user
---

# Guest access

A B2B guest is an external identity given an object inside the target tenant. Guests are often under-restricted: they can enumerate users, groups, and applications, and in permissive tenants reach data and even escalate toward member-equivalent access. A guest foothold is a cheap way inside a tenant you do not own.

## Enumerate as a guest

```bash
# authenticated as a guest, enumerate the directory (default guest permissions often allow this)
az rest --method GET --url "https://graph.microsoft.com/v1.0/users?\$top=999"
az rest --method GET --url "https://graph.microsoft.com/v1.0/groups"
# roadrecon runs a full guest-scoped dump
roadrecon gather
```

## Exploitation notes

- Default guest permissions in many tenants allow broad read of users, groups, and apps, which maps the whole attack surface from outside.
- Guests can own or be added to groups and apps, which opens [group](../groups/index.md) and [application](../applications/index.md) edges that escalate toward member access.
- Invitation is low-friction: an accepted invite plants the guest object; redemption can sometimes be pre-empted to take the account.

## Tools

- **ROADtools** (`roadrecon`): guest-scoped directory enumeration.
- **az cli** / **Graph**.

## References

- [dirkjanm.io: B2B guest abuse](https://dirkjanm.io/)
- [HackTricks Cloud: guest access](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
- [Microsoft: B2B collaboration](https://learn.microsoft.com/entra/external-id/what-is-b2b)
