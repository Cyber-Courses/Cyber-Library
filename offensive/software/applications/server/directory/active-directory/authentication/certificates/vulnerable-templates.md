---
title: "Vulnerable templates: ESC1, ESC2, ESC3, ESC13, ESC15"
description: "Abusing misconfigured AD CS certificate templates to enrol for a certificate that authenticates as a privileged account: enrollee-supplied subject (ESC1), overbroad EKUs (ESC2), enrollment agents (ESC3), OID group links (ESC13), and application-policy injection (ESC15)."
keywords:
  - ESC1
  - ESC2
  - ESC3
  - ESC13
  - ESC15
---

# Vulnerable templates

A certificate template defines what a certificate is for and who may request it. When a template combines a **low-privilege enrolment right** with settings that let the requester choose *who the certificate is for* or *what it can do*, any such user can enrol for a certificate that authenticates as a privileged account. These are the most direct AD CS escalations.

## ESC1: enrollee-supplied subject

The classic case. A template that (1) grants enrolment to low-privileged users, (2) has a **client-authentication EKU** (Client Authentication, Smart Card Logon, PKINIT Client Authentication, or Any Purpose), and (3) lets the **enrollee supply the subject** (`CT_FLAG_ENROLLEE_SUPPLIES_SUBJECT`). Two issuance gates must also be open: **manager approval disabled** (otherwise the request is held pending a certificate manager) and **no authorized signatures required** (otherwise the request needs a co-signing enrollment-agent certificate). With all of these, you request a certificate and specify a **SAN** naming a Domain Admin:

```bash
certipy req -u user@example.local -p pass -ca <ca-name> -template <vuln-template> \
  -upn administrator@example.local
# -> administrator.pfx, then authenticate as the DA
certipy auth -pfx administrator.pfx -dc-ip <dc>
```

## ESC2 and ESC3: overbroad purposes and enrollment agents

- **ESC2**: a template with the **Any Purpose** EKU (or no EKU at all, a SubCA template). The issued certificate can be used for client authentication even though nothing says "client auth", so an enrollee-supplied-subject or mapping weakness turns it into impersonation.
- **ESC3**: a template granting the **Certificate Request Agent** EKU. You enrol for an enrollment-agent certificate, then use it to enrol **on behalf of** another user against a template that accepts enrollment agents:

```bash
certipy req -u user@example.local -p pass -ca <ca> -template <agent-template>
certipy req -u user@example.local -p pass -ca <ca> -template User \
  -on-behalf-of 'EXAMPLE\administrator' -pfx agent.pfx
```

## ESC13: issuance policy linked to a group

A template with an **issuance policy** whose OID is linked to an AD group (`msDS-OIDToGroupLink`). A certificate issued from that template grants the holder the privileges of the linked group, so enrolling (even for yourself) yields a token with that group membership, without any subject trickery.

## ESC15: application-policy injection (EKUwu)

On **version 1** schema templates served by a **CA that is still vulnerable to the application-policy injection flaw** (patched CAs ignore injected policies), the requester can inject **application policies** into the CSR that the CA includes in the certificate, even when the template's EKU would not permit client authentication. Adding a Client Authentication application policy produces an auth-capable certificate from a template that looked safe. Naming another principal still requires the template to let the enrollee supply the subject; otherwise the injected policy only upgrades a certificate for yourself:

```bash
certipy req -u user@example.local -p pass -ca <ca> -template <v1-template> \
  -application-policies 'Client Authentication' -upn administrator@example.local
```

## Exploitation notes

- ESC1 and ESC15 both end in a certificate naming a Domain Admin; follow with `certipy auth` (PKINIT) and, if useful, [UnPAC-the-hash](../kerberos/unpac-the-hash.md) to recover the target's NT hash.
- ESC3 is valuable where no enrollee-supplied-subject template exists: the enrollment-agent path reaches privileged users through a normally benign template.
- The SID security extension (see [certificate mapping](certificate-mapping.md)) can block naive SAN impersonation on patched domains, which is why ESC1 is often paired with ESC9/ESC16 or shifted to ESC15.

## Tools

- **Certipy** (`req`, `auth`, `-on-behalf-of`, `-application-policies`): enrol and authenticate for all of these.
- **Certify / ForgeCert**: Windows-side enrolment and forging.

## References

- [SpecterOps: Certified Pre-Owned (ESC1-ESC3)](https://specterops.io/blog/2021/06/17/certified-pre-owned/)
- [TrustedSec: EKUwu, not just another AD CS ESC (ESC15)](https://trustedsec.com/blog/ekuwu-not-just-another-ad-cs-esc)
- [Certipy (ly4k)](https://github.com/ly4k/Certipy)
