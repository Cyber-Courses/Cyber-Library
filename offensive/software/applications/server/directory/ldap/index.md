---
title: "LDAP directories: attacking the directory protocol and its services"
description: "Offensive techniques against LDAP directory services (AD, OpenLDAP, 389 Directory Server): anonymous and authenticated enumeration, credentials exposed in attributes, and LDAP signing and channel-binding weaknesses."
keywords:
  - LDAP
  - anonymous bind
  - OpenLDAP
  - LDAP signing
  - channel binding
---

# LDAP

LDAP is the query protocol directories speak, whether the directory is Active Directory, OpenLDAP, 389 Directory Server, or an appliance's embedded directory. This section covers the attacks that target the LDAP service itself rather than a specific product: reading the directory with or without credentials, harvesting secrets that administrators store in object attributes, and the transport weaknesses (missing signing and channel binding) that make the service relay-able.

These apply across products. The AD-specific object model and attack chains live under [Active Directory](../active-directory/index.md); this section is the home for the protocol itself and for non-AD directories.

## Where it breaks

- **Over-permissive reads.** Directories frequently allow anonymous or null binds, and even authenticated reads expose far more than intended, handing over the user and group inventory and the directory's structure.
- **Secrets in attributes.** Administrators stash passwords in `description`, `userPassword`, `info`, and custom fields, and AD stores machine and service passwords in LAPS and gMSA attributes readable by specific principals.
- **Weak transport.** LDAP that does not require signing or channel binding can be relayed, turning a coerced or captured authentication into directory writes.

## Sections

- **[Protocol](protocol/index.md)**: the implementation-agnostic attacks that work against any LDAP directory, including anonymous and authenticated enumeration, credentials stored in attributes, and signing and channel-binding weaknesses.

Per-implementation coverage for OpenLDAP, 389 Directory Server, and eDirectory is organized alongside Protocol as those pages are written.

## References

- The Hacker Recipes: LDAP
- RFC 4511: Lightweight Directory Access Protocol (LDAP)
