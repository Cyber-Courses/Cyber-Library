---
title: "ldap:// and ldaps:// in SSRF: Java and enterprise URL handlers, blind fetches, and environment-specific chains"
description: LDAP URL handlers that issue directory or generic TCP connections from server-side URL fetchers.
keywords:
  - SSRF
  - ldap
---

# ldap:// (SSRF)

## Context

Some Java and enterprise clients resolve `ldap://` and `ldaps://` through JNDI-style code paths (distinct from log4j-specific issues). A fetch to `ldap://attacker/` can be used for blind SSRF, or in older chains for remote classloading where the environment is vulnerable, treat as **environment-specific** and **lab-only**.

## Theory

Scheme allowlists often forget `ldap` when they block `http` metadata IPs.

## Practice

- In a controlled lab, compare application behavior for `ldap://127.0.0.1:389/` vs malformed host.

## Tools

- **Burp Suite**
