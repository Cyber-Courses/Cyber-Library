---
title: "LDAP protocol attacks"
description: "Implementation-agnostic LDAP attacks that work against any directory server: anonymous and authenticated binds and enumeration, bind brute force, credentials stored in object attributes, and LDAP signing and channel-binding weaknesses."
keywords:
  - LDAP
  - anonymous bind
  - LDAP enumeration
  - LDAP signing
  - channel binding
---

# LDAP protocol attacks

These attacks target the LDAP service itself and work against any directory that speaks the protocol, whether it is Active Directory, OpenLDAP, 389 Directory Server, or an appliance's embedded directory. They turn on reading the directory with or without credentials, harvesting the secrets administrators leave in object attributes, and the transport weaknesses that let a captured authentication be relayed into the directory.

The product-specific object models and attack chains live elsewhere: [Active Directory](../../active-directory/index.md) for AD, and the per-implementation sections under [LDAP](../index.md) for OpenLDAP, 389 Directory Server, and eDirectory.

## Pages

- **[Anonymous bind and enumeration](anonymous-bind-and-enumeration.md)**: reading a directory with a null or anonymous bind.
- **[Authenticated enumeration](authenticated-enumeration.md)**: pulling the full object set with any valid bind.
- **[Credentials in attributes](credentials-in-attributes.md)**: passwords and managed secrets stored in object fields.
- **[LDAP signing and channel binding](signing-and-channel-binding.md)**: the transport weaknesses that enable relay.

## References

- [The Hacker Recipes: LDAP](https://www.thehacker.recipes/ad/recon/ldap)
- [RFC 4511: Lightweight Directory Access Protocol (LDAP): The Protocol](https://www.rfc-editor.org/rfc/rfc4511)
