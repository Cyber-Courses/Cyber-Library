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

A template flagged `CT_FLAG_NO_SECURITY_EXTENSION` produces certificates **without** the SID binding. This only helps where a domain controller still permits weak mapping. Before full enforcement, `StrongCertificateBindingEnforcement` values `0` (disabled) and `1` (compatibility) allowed a certificate without the SID extension to fall back to **UPN** mapping, while `2` rejected it. Microsoft moved updated domain controllers to full enforcement in February 2025 and stopped supporting the registry override after the September 9, 2025 security update. Treat the Kerberos form of ESC9 as a legacy, unpatched, or deliberately unsupported configuration rather than a current default. On such a DC, if you can control a principal's UPN (for example you have write over a low-privileged user and can set its UPN to a target's), you enroll with that account, set its UPN to the victim, and the issued certificate maps to the victim:

```bash
# With GenericWrite over 'puppet': set its UPN to the target, enroll, then restore
certipy account update -u user@example.local -p pass -user puppet -upn administrator@example.local
certipy req -u puppet@example.local -p pass2 -ca <ca> -template <esc9-template>
certipy account update -u user@example.local -p pass -user puppet -upn puppet@example.local
certipy auth -pfx administrator.pfx -domain example.local   # maps to administrator
```

## ESC10: weak mapping configuration

ESC10 covers weak certificate-mapping settings on the domain controller:

- **Kerberos (legacy systems)**: `StrongCertificateBindingEnforcement = 0` disabled the SID-binding requirement, so UPN mapping was trusted. Updated domain controllers no longer support this override after September 9, 2025.
- **Schannel**: `CertificateMappingMethods = 0x4` enables UPN mapping for Schannel.

Where the relevant weak mapping is still active, the UPN-swap trick can work against standard templates, widening ESC9 from one template to the whole CA. Test Kerberos and Schannel separately because their mapping controls and authentication paths differ.

## ESC14: attacker-written explicit mapping

Accounts can carry **explicit** certificate mappings in the `altSecurityIdentities` attribute. If you have write over that attribute on a target account, you add a mapping to a certificate you control, then authenticate as the target with it:

```bash
# Add an explicit mapping tying your certificate to the victim, then authenticate
certipy account update -u user@example.local -p pass -user victim \
  -alt-security-identities '<mapping-string-for-your-cert>'
```

This is a pure ACL abuse: a write primitive over `altSecurityIdentities` (from BloodHound) becomes authentication as the target.

## Exploitation notes

- These pair with an enrollment path: ESC9 needs a template you can enroll from, ESC14 needs a certificate to map; the mapping weakness is what turns the certificate into the *wrong* identity.
- On fully updated domain controllers, Kerberos rejects certificates that lack a strong mapping. Do not report ESC9 or Kerberos ESC10 from template or registry observations alone; prove that authentication succeeds through the intended protocol.
- ESC10's Schannel variant enables authentication over LDAPS/HTTPS rather than Kerberos, useful where PKINIT is constrained.

## Tools

- **Certipy** (`account update`, `req`, `auth`): UPN swaps, altSecurityIdentities writes, and PKINIT.
- **PowerView / native LDAP**: set `userPrincipalName` and `altSecurityIdentities`.

## References

- [Certipy wiki: privilege escalation (ESC9/ESC10)](https://github.com/ly4k/Certipy/wiki/06-%E2%80%90-Privilege-Escalation)
- [Microsoft KB5014754: certificate-based authentication changes](https://support.microsoft.com/en-us/servicing/os/windows-server/2022/05/kb5014754-certificate-based-authentication-changes-on-windows-domain-controllers)
- [SpecterOps: BloodHound ADCSESC9 edge](https://bloodhound.specterops.io/resources/edges/adcs-esc9a)
- [SpecterOps: Certified Pre-Owned](https://specterops.io/blog/2021/06/17/certified-pre-owned/)
