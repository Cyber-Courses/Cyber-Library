---
title: "AD CS enumeration: finding CAs, templates, and weaknesses"
description: "Enumerating Active Directory Certificate Services from a domain account: locating enterprise CAs, published certificate templates, enrolment rights, and the misconfigurations (ESC conditions) that make certificates abusable."
keywords:
  - certipy find
  - certificate template
  - enterprise CA
  - pKIEnrollmentService
  - ESC
---

# AD CS enumeration

AD CS configuration lives in the **Configuration partition** of the directory, readable by any authenticated user. That means a single domain account is enough to inventory every CA, every published template, who may enrol, and which templates or CAs are misconfigured, before touching anything. Enumeration decides which escalation (if any) is available, so it is always the first step.

## What to collect

- **Enterprise CAs**: the `pKIEnrollmentService` objects under `CN=Enrollment Services`, giving each CA's name, DNS host, and the templates it publishes.
- **Certificate templates**: `pKICertificateTemplate` objects, with their EKUs, enrolment flags, and the ACL that says who can enrol.
- **The NTAuthCertificates** object: which CAs are trusted for authentication in the forest.
- **Enrolment rights and object ACLs**: who can enrol on each template, and who can *write* to templates, CAs, or PKI containers.

## Running the enumeration

```bash
# Certipy: collect everything and flag vulnerable configurations
certipy find -u user@example.local -p pass -dc-ip <dc> -stdout -vulnerable

# NetExec from Linux: find CAs/templates, or run Certipy's triage inline
nxc ldap <dc> -u user -p pass -M adcs
nxc ldap <dc> -u user -p pass -M certipy-find

# BloodHound (with AD CS collection) graphs CA/template relationships and ESC paths
# Certutil, on a domain-joined host
certutil -config - -ping
certutil -template
```

`certipy find -vulnerable` is the workhorse: it reads the templates and CA settings and labels each applicable **ESC** condition, so you can go straight to the matching technique.

## Reading the results

The conditions that make a template or CA abusable, and the page that covers each:

- Enrollee-supplied subject + an auth EKU + low-priv enrol: **[ESC1](vulnerable-templates.md)**.
- Any Purpose / no EKU, or Enrollment Agent templates: **[ESC2/ESC3](vulnerable-templates.md)**.
- Missing security extension, or weak account mapping: **[ESC9/ESC10/ESC14](certificate-mapping.md)**.
- Writable template/CA/PKI object: **[ESC4/ESC5/ESC7](access-control.md)**.
- CA SAN-attribute flag, or disabled SID binding: **[ESC6/ESC16](ca-configuration.md)**.
- Web or RPC enrolment reachable for relay: **[ESC8/ESC11](relay-to-adcs.md)**.

## Exploitation notes

- Enumeration is low-privilege and low-noise (directory reads plus a CA ping), so run it early with any valid credential.
- A template is only exploitable if you also hold **enrolment rights** on it, directly or through a group; cross-check the template ACL against your group memberships.
- The CA host DNS name and its enrolment endpoints (web enrolment, CES) feed the relay techniques, so record them even when a template path looks easier.

## Tools

- **Certipy** (`find -vulnerable`): the primary AD CS enumeration and triage tool.
- **NetExec (`nxc`) `-M adcs` / `-M certipy-find`**: CA and template discovery, and Certipy triage, over LDAP from Linux.
- **BloodHound** (AD CS collection): graphs CA/template/enrolment relationships.
- **certutil / PSPKIAudit**: native and PowerShell enumeration.

## References

- [Certipy wiki: privilege escalation (ESC triage)](https://github.com/ly4k/Certipy/wiki/06-%E2%80%90-Privilege-Escalation)
- [NetExec adcs module source](https://github.com/Pennyw0rth/NetExec/blob/main/nxc/modules/adcs.py)
- [SpecterOps: Certified Pre-Owned](https://specterops.io/blog/2021/06/17/certified-pre-owned/)
