---
title: "Ownership and ACL rewrite: WriteOwner and WriteDacl"
description: "Escalating a partial Active Directory write to full control: WriteDacl to add an ACE granting yourself rights, and WriteOwner to take ownership, plus the modern Owner Rights and BlockOwnerImplicitRights limits that stop ownership from implying WriteDacl on hardened domains."
keywords:
  - WriteDacl
  - WriteOwner
  - owneredit
  - dacledit
  - Owner Rights
---

# Ownership and ACL rewrite

`WriteDacl` and `WriteOwner` are the meta-rights: they do not act on the object's data, they let you **change who has rights over it**. With either, you grant yourself whatever right you lack (`GenericAll`, the DCSync replication rights, and so on) and proceed from there. `WriteDacl` does this directly; `WriteOwner` does it in two steps, and on modern domains that second path no longer always works.

## WriteDacl: grant yourself rights

`WriteDacl` lets you add an ACE to the object's DACL. Add one granting a principal you control full control, or just the replication rights for [DCSync](../authentication/credentials/ntds-and-dcsync.md):

```bash
# Impacket dacledit.py: write a FullControl ACE for a controlled principal
dacledit.py -action write -rights FullControl -principal 'user' -target 'victim' example.local/user:pass
# or only the DCSync rights, when the target is the domain object
dacledit.py -action write -rights DCSync -principal 'user' -target-dn 'DC=example,DC=local' example.local/user:pass

# bloodyAD equivalents
bloodyAD --host <dc> -d example.local -u user -p pass add genericAll 'victim' 'user'
bloodyAD --host <dc> -d example.local -u user -p pass add dcsync 'user'
```

## WriteOwner: take ownership, then rewrite

`WriteOwner` lets you set the object's **owner**. Historically an owner implicitly held `READ_CONTROL` and `WRITE_DAC`, so taking ownership let you rewrite the DACL and grant yourself control:

```bash
# Set a controlled principal as owner
owneredit.py -action write -new-owner 'user' -target 'victim' example.local/user:pass
bloodyAD --host <dc> -d example.local -u user -p pass set owner 'victim' 'user'
# then rewrite the DACL as above (dacledit / bloodyAD add genericAll)
```

```powershell
Set-DomainObjectOwner -Identity victim -OwnerIdentity user   # PowerView
```

## The Owner Rights limit (why WriteOwner can be a dead end now)

The implicit-owner-rights behaviour has been tightened, and up-to-date environments may not grant a new owner `WRITE_DAC` at all:

- The **`OWNER RIGHTS` SID (`S-1-3-4`)**: if the object's DACL contains an ACE for this SID, it **replaces** the owner's implicit rights with exactly what that ACE allows. An `OWNER RIGHTS` ACE granting only read effectively removes the owner's implicit `WRITE_DAC`.
- **`BlockOwnerImplicitRights`** (the **28th character of `dsHeuristics`**): when set, an owner that is **not** a member of Domain Admins or Enterprise Admins gets only limited implicit rights, not `WRITE_DAC`. So on a hardened domain, taking ownership as a normal principal does not let you rewrite the DACL.

BloodHound reflects this with the **`WriteOwnerLimitedRights`** / **`OwnsLimitedRights`** edges: ownership is still writable, but the follow-on DACL rewrite is not guaranteed. Before relying on a `WriteOwner` path, read `dsHeuristics` and the target's `OWNER RIGHTS` ACEs; where implicit rights are blocked, you need a direct `WriteDacl` edge instead, and `WriteOwner` alone is not enough.

## Exploitation notes

- Prefer a direct `WriteDacl` edge when you have it; it avoids the Owner Rights question entirely.
- Grant the **minimum** you need (DCSync on the domain, or full control on one object) rather than broad rights, and **restore** the original owner/DACL afterwards: `dacledit.py` and `owneredit.py` both support reading and restoring so you can revert.
- Editing an owner or DACL generates directory-service change events on the handling DC, so batch the change-use-revert quickly.

## Tools

- **Impacket** (`dacledit.py` read/write/backup/restore, `owneredit.py`): ACE and owner edits from Linux.
- **bloodyAD** (`set owner`, `add genericAll`, `add dcsync`): one-shot grants from Linux.
- **PowerView** (`Set-DomainObjectOwner`, `Add-DomainObjectAcl`): on-host edits.

## References

- SpecterOps: Do You Own Your Permissions, or Do Your Permissions Own You? (Owner Rights, BlockOwnerImplicitRights)
- The Hacker Recipes: WriteDacl and WriteOwner
