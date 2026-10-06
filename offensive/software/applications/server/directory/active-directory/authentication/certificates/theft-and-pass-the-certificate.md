---
title: "Theft and pass-the-certificate: stealing and using certificates"
order: 5
description: "Stealing certificates and private keys from Windows hosts (user and machine stores, DPAPI-protected keys, exported PFX) and authenticating with them via PKINIT and Schannel (pass-the-certificate), including recovering the NT hash."
keywords:
  - pass the certificate
  - certificate theft
  - PKINIT
  - DPAPI
  - PFX
---

# Theft and pass-the-certificate

Not every certificate attack requires a misconfiguration. Certificates and their private keys already exist on hosts, in user and machine stores, and a valid authentication certificate is a durable credential: it is not invalidated by a password change and often lasts a year or more. **Stealing** one and **authenticating with it** (pass-the-certificate) is a clean lateral-movement and persistence primitive.

## Stealing certificates

- **User and machine certificate stores**: exportable certificates with their private keys can be dumped to PFX, from memory or the store, on a compromised host.
- **DPAPI-protected keys**: private keys are protected by [DPAPI](../credentials/dpapi-secrets.md); with the user's master key (or the domain backup key) they decrypt offline.
- **Machine certificates**: a host's own certificate authenticates as the **machine account**, useful for silver tickets and RBCD.

```bash
# Mimikatz: export certificates and keys from the stores
crypto::capi & crypto::certificates /export
# Certipy: collect certificates and DPAPI-protected keys from a host
certipy cert -export ...
```

## Pass-the-certificate

A stolen or issued certificate authenticates two ways:

- **PKINIT** (Kerberos): exchange the certificate for a **TGT**, then move with the ticket:

```bash
certipy auth -pfx victim.pfx -dc-ip <dc>          # PKINIT -> TGT (and NT hash via UnPAC)
gettgtpkinit.py -cert-pfx victim.pfx example.local/victim victim.ccache
```

- **Schannel** (LDAPS/HTTPS): bind to LDAP over TLS with the client certificate, useful where PKINIT is unavailable but Schannel mapping is enabled ([ESC10](certificate-mapping.md)):

```bash
certipy auth -pfx victim.pfx -ldap-shell -dc-ip <dc>
```

## Recovering the NT hash

PKINIT returns the account's NT hash alongside the TGT via [UnPAC-the-hash](../kerberos/unpac-the-hash.md), so a stolen certificate becomes a **reusable hash**, converting a time-limited certificate into a credential that drives [pass-the-hash](../ntlm/pass-the-hash.md) indefinitely.

## Exploitation notes

- A certificate survives the owner's **password reset**, so a stolen or attacker-enrolled certificate is persistence: it keeps authenticating until it expires or is revoked.
- Machine certificates authenticate as `HOST$` and feed [silver tickets](../kerberos/forged-tickets.md) and [RBCD](../kerberos/delegation/resource-based-constrained.md).
- Enrolling a certificate for *yourself* now, while you have access, is a deliberate persistence move (certificate account persistence), independent of any ESC.

## Tools

- **Certipy** (`cert`, `auth`, `-ldap-shell`): export, PKINIT, and Schannel authentication.
- **Mimikatz** (`crypto::certificates`): export certificates and keys from Windows stores.
- **PKINITtools**: explicit PKINIT and UnPAC.

## References

- [Certipy (ly4k): auth, shadow, and certificate theft](https://github.com/ly4k/Certipy)
- [PKINITtools (dirkjanm): PKINIT and UnPAC](https://github.com/dirkjanm/PKINITtools)
- [SpecterOps: Certified Pre-Owned (theft and persistence)](https://specterops.io/blog/2021/06/17/certified-pre-owned/)
