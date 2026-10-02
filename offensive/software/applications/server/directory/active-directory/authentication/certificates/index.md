---
title: "Certificates: abusing AD Certificate Services"
description: "Abusing Active Directory Certificate Services for authentication and privilege escalation: enrolling for certificates that impersonate privileged accounts, abusing CA configuration and access control, relaying to enrollment, and stealing certificates."
keywords:
  - AD CS
  - ESC1
  - certificate template
  - certificate authority
  - pass the certificate
---

# Certificates

Active Directory Certificate Services (AD CS) issues certificates that Windows accepts as **authentication material**: a certificate with a client-authentication purpose can be used with PKINIT to obtain a Kerberos TGT, so holding the right certificate *is* holding the identity it names. AD CS is widely deployed, frequently misconfigured, and rarely monitored, which is why the "Certified Pre-Owned" family of techniques turns it into one of the most reliable escalation paths to Domain Admin.

## Why a certificate is an identity

- A certificate binds a **subject** (often a UPN or, since the security-extension update, a SID) to a key pair.
- A domain controller maps that subject to an account and issues a TGT (PKINIT) or authenticates a TLS session (Schannel).
- So if you can obtain a certificate that **names a privileged account**, or weaken how the mapping is enforced, you authenticate as that account, no password or hash required.

Every AD CS attack is a variation on this: get a certificate you should not have (template, CA, or relay abuse), or break the mapping that is supposed to tie a certificate to its rightful owner.

## The classes of abuse

- **Template misconfiguration** (ESC1, ESC2, ESC3, ESC13, ESC15): a template lets a low-privileged user enrol for a certificate that authenticates as someone else.
- **Certificate mapping** (ESC9, ESC10, ESC14): weakened or missing SID binding lets a certificate map to a more privileged account than it names.
- **Access control** (ESC4, ESC5, ESC7): write access over templates, PKI objects, or CA roles is turned into enrolment.
- **CA configuration** (ESC6, ESC16): a CA flag that lets any request specify its own subject, or that disables the SID binding domain-wide.
- **Enrolment relay** (ESC8, ESC11): NTLM relay to the CA's HTTP or RPC enrolment endpoints.

## Pages

- **[Enumeration](enumeration.md)**: finding CAs, templates, and vulnerable conditions.
- **[Vulnerable templates](vulnerable-templates.md)**: ESC1, ESC2, ESC3, ESC13, ESC15.
- **[Certificate mapping](certificate-mapping.md)**: ESC9, ESC10, ESC14.
- **[Access control](access-control.md)**: ESC4, ESC5, ESC7.
- **[CA configuration](ca-configuration.md)**: ESC6, ESC16.
- **[Relay to AD CS](relay-to-adcs.md)**: ESC8, ESC11.
- **[Theft and pass-the-certificate](theft-and-pass-the-certificate.md)**: stealing certificates and authenticating with them.

## References

- [SpecterOps: Certified Pre-Owned](https://specterops.io/blog/2021/06/17/certified-pre-owned/)
- [Certipy (ly4k): AD CS enumeration and abuse](https://github.com/ly4k/Certipy)
- [Microsoft: Active Directory Certificate Services](https://learn.microsoft.com/en-us/windows-server/identity/ad-cs/)
