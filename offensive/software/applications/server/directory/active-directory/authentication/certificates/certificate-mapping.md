---
title: "Certificate mapping: ESC9, ESC10, ESC14"
order: 2
description: "Abusing weak certificate-to-account mapping in Active Directory: the missing SID security extension (ESC9), weak Kerberos and Schannel mapping registry settings (ESC10), and attacker-written explicit mappings via altSecurityIdentities (ESC14)."
keywords:
  - ESC9
  - ESC10
  - ESC14
  - altSecurityIdentities
  - certificate mapping
---

# Certificate mapping

When an account authenticates with a certificate, the domain controller must decide **which account** the certificate represents. The SID security extension (`szOID_NTDS_CA_SECURITY_EXT`) was introduced to bind a certificate to its owner's SID so the subject cannot be impersonated. These techniques abuse mappings where that binding is **missing or weak**, so a certificate maps to a more privileged account than it should.

## ESC9: no security extension

A template flagged `CT_FLAG_NO_SECURITY_EXTENSION` produces certificates **without** the SID binding. This only helps where the DC still permits weak mapping: with `StrongCertificateBindingEnforcement` at `0` (disabled) or `1` (compatibility), a certificate lacking the SID extension falls back to **UPN** mapping, but at `2` (full enforcement) it is **rejected** outright. On a weakly-mapping DC, if you can control a principal's UPN (for example you have write over a low-privileged user and can set its UPN to a target's), you enrol with that account, set its UPN to the victim, and the issued certificate maps to the victim:

```bash
# With GenericWrite over 'puppet': set its UPN to the target, enrol, then restore
certipy account update -u user@example.local -p pass -user puppet -upn administrator@example.local
certipy req -u puppet@example.local -p pass2 -ca <ca> -template <esc9-template>
certipy account update -u user@example.local -p pass -user puppet -upn puppet@example.local
certipy auth -pfx administrator.pfx -domain example.local   # maps to administrator
```

## ESC10: weak mapping configuration

ESC10 is the same impersonation enabled by **registry** settings on the DC rather than a template flag:

- **Kerberos**: `StrongCertificateBindingEnforcement = 0` disables the SID-binding requirement, so UPN mapping is trusted.
- **Schannel**: `CertificateMappingMethods = 0x4` enables UPN mapping for Schannel.

Either one lets the UPN-swap trick above work against standard templates, widening ESC9 from one template to the whole CA.

## ESC14: attacker-written explicit mapping

Accounts can carry **explicit** certificate mappings in the `altSecurityIdentities` attribute. If you have write over that attribute on a target account, you add a mapping to a certificate you control, then authenticate as the target with it:

```bash
# Add an explicit mapping tying your certificate to the victim, then authenticate
certipy account update -u user@example.local -p pass -user victim \
  -alt-security-identities '<mapping-string-for-your-cert>'
```

This is a pure ACL abuse: a write primitive over `altSecurityIdentities` (from BloodHound) becomes authentication as the target.

## Exploitation notes

- These pair with an enrolment path: ESC9 needs a template you can enrol from, ESC14 needs a certificate to map; the mapping weakness is what turns the certificate into the *wrong* identity.
- Where the SID extension is enforced (patched, strong binding), naive SAN impersonation ([ESC1](vulnerable-templates.md)) fails and these mapping gaps or [ESC15](vulnerable-templates.md) become the way through.
- ESC10's Schannel variant enables authentication over LDAPS/HTTPS rather than Kerberos, useful where PKINIT is constrained.

## Tools

- **Certipy** (`account update`, `req`, `auth`): UPN swaps, altSecurityIdentities writes, and PKINIT.
- **PowerView / native LDAP**: set `userPrincipalName` and `altSecurityIdentities`.

## References

- [Certipy wiki: privilege escalation (ESC9/ESC10)](https://github.com/ly4k/Certipy/wiki/06-%E2%80%90-Privilege-Escalation)
- [SpecterOps: BloodHound ADCSESC9 edge](https://bloodhound.specterops.io/resources/edges/adcs-esc9a)
- [SpecterOps: Certified Pre-Owned](https://specterops.io/blog/2021/06/17/certified-pre-owned/)
