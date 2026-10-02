---
title: "Access control: ESC4, ESC5, ESC7"
description: "Abusing write access over AD CS objects: reconfiguring a template through its ACL (ESC4), controlling CA or PKI objects and the CA host (ESC5), and abusing CA role rights ManageCA and ManageCertificates (ESC7)."
keywords:
  - ESC4
  - ESC5
  - ESC7
  - ManageCA
  - certificate template ACL
---

# Access control

Several AD CS escalations are not about a standing misconfiguration but about **write access** to the objects that control certification. A permission over a template, a CA object, or a CA role lets you *create* the vulnerable condition yourself, then enrol. These are the paths BloodHound surfaces as writable AD CS objects.

## ESC4: writable template

If you have write access (`GenericWrite`, `GenericAll`, `WriteDacl`, `WriteOwner`) over a **certificate template** object, you can rewrite its settings to make it [ESC1](vulnerable-templates.md)-vulnerable: enable enrollee-supplied subject, add a client-auth EKU, and grant yourself enrolment. Then enrol as a Domain Admin and, ideally, revert the template:

```bash
# Certipy can reconfigure a template to be vulnerable, enrol, and restore it
certipy template -u user@example.local -p pass -template <template> -write-default-configuration
# ... enrol with -upn administrator@example.local, then restore the saved config
```

## ESC5: control of CA objects or the CA host

ESC5 is the broad category of controlling objects the PKI depends on, outside the templates themselves:

- The **CA computer object** or the CA server host (local admin on the CA lets you read its private key and issue arbitrary certificates).
- The **CA's AD objects** (the `pKIEnrollmentService` / `certificationAuthority` objects, the PKI containers).
- The **NTAuthCertificates** object, which lists CAs trusted for authentication; writing it can add a rogue CA to the forest's trust.

Control of any of these is equivalent to control of certificate issuance, so it is treated as a domain-escalation primitive.

## ESC7: CA role rights

The CA itself has two powerful roles whose rights can be delegated:

- **ManageCA** (CA administrator): can change CA configuration, including setting the `EDITF_ATTRIBUTESUBJECTALTNAME2` flag that enables [ESC6](ca-configuration.md), and can enable certificate-manager approvals.
- **ManageCertificates** (certificate manager): can **approve pending requests**, so a request that would be held for approval can be approved by the same attacker.

Holding both (or ManageCA to grant yourself ManageCertificates) lets you submit a request for a privileged identity and approve it:

```bash
# Certipy: use ManageCA/ManageCertificates to issue despite restrictions
certipy ca -u user@example.local -p pass -ca <ca> -add-officer user       # grant cert-manager
certipy req ... -template <template>                                        # submit
certipy ca -u user@example.local -p pass -ca <ca> -issue-request <id>       # approve
```

## Exploitation notes

- ESC4 is the cleanest: one writable template becomes ESC1, so hunt for write access over templates during [ACL enumeration](../../dacl/acl-enumeration.md).
- ESC7's ManageCA path can also **enable the web enrolment endpoint**, opening the door to [ESC8 relay](relay-to-adcs.md).
- Always restore a reconfigured template or CA setting afterward to limit disruption and keep the change inconspicuous.

## Tools

- **Certipy** (`template`, `ca`): reconfigure templates, manage CA roles, approve requests.
- **PSPKI / Certify**: Windows-side CA and template management.

## References

- SpecterOps: Certified Pre-Owned (ESC4, ESC5, ESC7)
- The Hacker Recipes: AD CS access control
