---
title: "LDAP signing and channel binding: the transport weaknesses that enable relay"
description: "How missing LDAP signing and channel binding (LDAPS EPA) let a coerced or captured authentication be relayed to the directory, and how to determine a directory's signing posture before attempting a relay."
keywords:
  - LDAP signing
  - channel binding
  - EPA
  - NTLM relay to LDAP
  - LDAPS
---

# LDAP signing and channel binding

Two transport protections decide whether authentication to a directory can be **relayed**: LDAP signing (on plain LDAP, 389) and channel binding (on LDAPS, 636). Where neither is enforced, an attacker who captures or coerces a victim's NTLM authentication can relay it to the directory and act as the victim, which is the gateway to writing directory objects (adding a computer, setting RBCD, granting DCSync). This page is the precondition check; the relay itself is covered under NTLM relay in Active Directory.

## Why it matters

A relay target is only useful if it accepts a relayed, unsigned authentication:

- **LDAP (389)** requires **signing** to be safe. If the directory does not enforce signing, relayed NTLM over plain LDAP is accepted, and the attacker performs directory operations as the relayed victim.
- **LDAPS (636)** is encrypted, so it cannot be signed in the same way; its protection is **channel binding** (EPA), which ties the authentication to the TLS channel. Without channel binding enforced, NTLM authentication captured elsewhere can be relayed to LDAPS.

For a long time both were off by default, so an unhardened domain is relay-able to LDAP, LDAPS, or both.

## Checking the posture

Determine what the directory enforces before planning a relay:

```bash
# NetExec reports LDAP signing and channel-binding enforcement
nxc ldap <dc> -u user -p pass -M ldap-checker

# Behavioral test: an unsigned simple bind that the server rejects indicates signing is required
ldapsearch -x -H ldap://<dc> -D 'EXAMPLE\user' -w pass -b '' -s base
```

Three states matter: signing not required (relay to 389 works), channel binding not required (relay to 636 works), and both enforced (LDAP relay is closed, pivot to a different relay target such as SMB or AD CS HTTP enrollment).

## Exploitation notes

- The check gates the attack: there is no point coercing authentication toward LDAP if the directory enforces both signing and channel binding.
- Even with LDAP hardened, the **AD CS web enrollment** endpoint (HTTP) is a common relay target that signing/channel-binding on LDAP does not protect, so a hardened directory does not close relay entirely.
- Signing/channel-binding posture is a domain-wide setting on the DCs, so one check against any DC characterizes the whole domain.

## Tools

- **NetExec (nxc) -M ldap-checker**: signing and channel-binding enforcement report.
- **ldapsearch**: manual unsigned-bind behavior check.
- **ntlmrelayx** (Impacket): the relay itself, once the posture is known.

## References

- The Hacker Recipes: NTLM relay
- Microsoft: LDAP channel binding and signing requirements
