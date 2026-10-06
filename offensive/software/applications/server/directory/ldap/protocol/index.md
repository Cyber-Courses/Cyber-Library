---
title: "LDAP protocol attacks"
description: "LDAP attacks that target the protocol rather than a product's object model: anonymous and authenticated binds and enumeration, credentials stored in object attributes, and LDAP signing and channel-binding weaknesses (the last specific to NTLM-authenticating directories such as Active Directory)."
keywords:
  - LDAP
  - anonymous bind
  - LDAP enumeration
  - LDAP signing
  - channel binding
---

# Protocol attacks

These attacks target the LDAP service itself rather than a product's object model. Reading the directory with or without credentials, and harvesting the secrets administrators leave in object attributes, work against any directory that speaks the protocol, whether Active Directory, OpenLDAP, 389 Directory Server, or an appliance's embedded directory. The transport weaknesses are narrower: LDAP signing and channel binding gate an NTLM relay into the directory, so they apply to Active Directory and other Windows-integrated directories, not to OpenLDAP or 389 Directory Server as such.

The product-specific object models and attack chains live elsewhere: [Active Directory](../../active-directory/index.md) for AD, and the per-implementation sections under [LDAP](../index.md) for OpenLDAP, 389 Directory Server, and eDirectory.

## Pages

- **[Anonymous bind and enumeration](anonymous-bind-and-enumeration.md)**: reading a directory with a null or anonymous bind.
- **[Authenticated enumeration](authenticated-enumeration.md)**: pulling the full object set with any valid bind.
- **[Credentials in attributes](credentials-in-attributes.md)**: passwords and managed secrets stored in object fields.
- **[LDAP signing and channel binding](signing-and-channel-binding.md)**: the transport weaknesses that enable relay.

## References

- [The Hacker Recipes: LDAP](https://www.thehacker.recipes/ad/recon/ldap)
- [RFC 4511: Lightweight Directory Access Protocol (LDAP): The Protocol](https://www.rfc-editor.org/rfc/rfc4511)
