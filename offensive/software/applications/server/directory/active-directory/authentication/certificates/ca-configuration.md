---
title: "CA configuration: ESC6 and ESC16"
description: "Abusing enterprise CA configuration flags: EDITF_ATTRIBUTESUBJECTALTNAME2 which lets any request specify its own SAN (ESC6), and a CA that disables the SID security extension for all issued certificates (ESC16)."
keywords:
  - ESC6
  - ESC16
  - EDITF_ATTRIBUTESUBJECTALTNAME2
  - security extension
  - certificate authority
---

# CA configuration

Some AD CS escalations come from the **certificate authority's own settings**, which apply to every certificate it issues regardless of the template. A single bad CA flag turns otherwise safe templates into escalation paths across the whole environment.

## ESC6: the SAN attribute flag

When a CA has `EDITF_ATTRIBUTESUBJECTALTNAME2` set, it honours a **subjectAltName supplied in the request** for *any* template, not just those configured for enrollee-supplied subject. That reproduces [ESC1](vulnerable-templates.md) everywhere: enrol for any client-auth template and attach a SAN naming a Domain Admin:

```bash
# With the CA flag set, specify the SAN on a normally safe template
certipy req -u user@example.local -p pass -ca <ca> -template User \
  -upn administrator@example.local
```

The flag is read during [enumeration](enumeration.md) and can be **set** by an attacker holding [ESC7](access-control.md) ManageCA rights, so ESC6 and ESC7 chain together.

Note the SID security extension complicates naive ESC6 on patched domains: where strong certificate binding is enforced, a SAN alone may not impersonate, and ESC6 is combined with a [mapping weakness](certificate-mapping.md).

## ESC16: security extension disabled on the CA

ESC16 is the CA-wide version of [ESC9](certificate-mapping.md): the CA is configured to **omit the SID security extension** from every certificate it issues (the extension OID is in the CA's disabled-extensions list). As with ESC9, this only yields impersonation where the DC still permits weak mapping (`StrongCertificateBindingEnforcement` at `0` or `1`); under full enforcement (`2`) a certificate without the SID binding is rejected. On a weakly-mapping DC the UPN-swap impersonation then works against the whole CA rather than a single template:

```bash
# Enrol after setting a controlled account's UPN to the victim; the issued cert
# lacks the SID binding, so it maps to the victim by UPN
certipy account update -u user@example.local -p pass -user puppet -upn administrator@example.local
certipy req -u puppet@example.local -p pass2 -ca <ca> -template User
```

## Exploitation notes

- ESC6 is a one-flag path to Domain Admin across all client-auth templates, which is why it is high-value when `certipy find` reports it.
- ESC6 set via ESC7 requires a **CA service restart** to take effect, which is noisy and may need the CA host; weigh it against quieter template paths.
- ESC16 turns every certificate into an ESC9 candidate, so it is as severe as a domain-wide mapping failure; pair with any enrolment right and a writable UPN.

## Tools

- **Certipy** (`req`, `ca`, `account update`): enrol with a chosen SAN, toggle CA flags (with ESC7), and UPN-swap.
- **certutil** (`-setreg`): native reading/setting of CA policy flags.

## References

- [SpecterOps: Certified Pre-Owned (ESC6)](https://specterops.io/blog/2021/06/17/certified-pre-owned/)
- [Certipy (ly4k)](https://github.com/ly4k/Certipy)
- [Microsoft: Active Directory Certificate Services](https://learn.microsoft.com/en-us/windows-server/identity/ad-cs/)
