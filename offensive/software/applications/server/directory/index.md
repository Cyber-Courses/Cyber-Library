---
title: "Directory services: attacking Active Directory and LDAP directories"
description: "Offensive techniques against directory services: the Active Directory attack surface (enumeration, authentication and credential abuse, ACLs, trusts, persistence) and generic LDAP directory attacks."
keywords:
  - active directory
  - LDAP
  - Entra ID
  - directory services
  - domain compromise
  - AD attacks
---

# Directory

A **directory service** stores and serves the identities, credentials, and authorization data an organization runs on. Compromising it is rarely about a single bug; it is about abusing the service's own features, protocols, and object permissions to move from a low-privileged foothold to control of the identity store, and with it the entire estate that trusts it.

This area splits by the directory product, because the attack surface is product-specific:

- **[Active Directory](active-directory/index.md)**, the Windows directory and its authentication ecosystem (Kerberos, NTLM, LDAP, AD CS). This is the dominant target in enterprise networks and the largest subtree.
- **[LDAP](ldap/index.md)**, the generic directory protocol as it appears on non-Windows directories (OpenLDAP, 389 Directory Server, and others) and as the LDAP layer that AD itself exposes.
- **[Entra ID](entra-id/index.md)**, Microsoft's cloud directory (formerly Azure AD): the tenant identity plane behind Microsoft 365 and Azure, attacked through sign-in, applications and service principals, directory roles, and devices.

## Scope and seams

Directory attacks sit at the service layer: you are attacking the directory *as a running service and data store*, not an application's query handling. Application-layer **LDAP injection** (user input concatenated into an LDAP filter in web code) is a different class and lives under [web code injection](../web/code/injection/database/ldap/index.md); this area cross-references it rather than repeating it.

Several named attacks that pass *through* a domain actually target separate products, so they live in their own server areas and are cross-referenced from Active Directory: the Netlogon protocol, the Print Spooler service, Exchange, and configuration-management platforms such as SCCM.

## Sections

- **[Active Directory](active-directory/index.md)**: enumeration, authentication and credential abuse, DACL abuse, Group Policy, trusts, and persistence.
- **[LDAP](ldap/index.md)**: anonymous and authenticated enumeration, credentials exposed in attributes, and signing and channel-binding weaknesses.
- **[Entra ID](entra-id/index.md)**: authentication and token abuse, application and service-principal takeover, directory-role and group escalation, device identity, and cross-tenant and guest access.

## References

- The Hacker Recipes: Active Directory
- PortSwigger / OWASP WSTG: Identity and authentication testing
