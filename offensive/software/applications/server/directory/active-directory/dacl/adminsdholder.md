---
title: "AdminSDHolder: DACL persistence through SDProp"
description: "Backdooring Active Directory by writing an ACE to the AdminSDHolder container, which SDProp stamps onto every protected group and account on its timer, re-granting attacker rights over Domain Admins and other protected principals even after they are removed."
keywords:
  - AdminSDHolder
  - SDProp
  - adminCount
  - protected groups
  - persistence
---

# AdminSDHolder

`AdminSDHolder` is a container (`CN=AdminSDHolder,CN=System,DC=...`) whose DACL is a **template**. A background process, the **Security Descriptor Propagator (SDProp)**, copies that DACL onto every **protected** object (the privileged groups and their members, marked `adminCount=1`) roughly every **60 minutes**, overwriting their individual DACLs. That design, meant to keep privileged objects consistently locked down, is a persistence primitive: write one ACE into the AdminSDHolder template and SDProp grants you rights over **all** protected principals, and re-grants them every cycle even if a defender strips them off the group.

## Planting the backdoor

With control of the AdminSDHolder object (owner, `WriteDacl`, or `GenericAll` over it, usually only reachable once you are already highly privileged), add an ACE granting a principal you control full control:

```bash
# Impacket dacledit.py: write a FullControl ACE into the AdminSDHolder template
dacledit.py -action write -rights FullControl -principal 'user' \
  -target-dn 'CN=AdminSDHolder,CN=System,DC=example,DC=local' example.local/admin:pass

# bloodyAD
bloodyAD --host <dc> -d example.local -u admin -p pass add genericAll \
  'CN=AdminSDHolder,CN=System,DC=example,DC=local' 'user'
```

```powershell
# PowerView
Add-DomainObjectAcl -TargetIdentity 'CN=AdminSDHolder,CN=System,DC=example,DC=local' `
  -PrincipalIdentity user -Rights All
```

Within one SDProp cycle, `user` holds `GenericAll` over Domain Admins, Administrators, and every other protected group and account, so it can reset their passwords, add members, or DCSync at will. To avoid waiting, SDProp can be triggered on the DC (the `RunProtectAdminGroupsTask` operation).

## Why it is durable

- The grant lives on the **template**, not the group, so removing your rights from Domain Admins is undone at the next cycle; the defender has to find and clean the AdminSDHolder DACL itself.
- It covers the whole **protected set** at once (anything with `adminCount=1`), so a single ACE is forest-domain-wide privileged access.
- It survives membership changes and password resets of the protected accounts, because it regrants the *permission*, not a credential.

## Exploitation notes

- This is **post-compromise persistence**: you already need high privilege to write AdminSDHolder, so it is about keeping access, not gaining it.
- Orphaned `adminCount=1` objects (accounts removed from a privileged group keep the flag) are a related tell and a hunting ground; the attribute lingers after demotion.
- Writing AdminSDHolder's DACL generates directory-service change events on the DC, so it is not stealthy at write time; its value is the durable, quietly-reapplied access afterwards.

## Tools

- **Impacket `dacledit.py`** / **bloodyAD**: write the template ACE from Linux.
- **PowerView** (`Add-DomainObjectAcl`): on-host, and `Get-DomainObjectAcl` to read the template back.

## References

- The Hacker Recipes: AdminSDHolder
- SpecterOps: An ACE Up the Sleeve (AdminSDHolder backdoors)
