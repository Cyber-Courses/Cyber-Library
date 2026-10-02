---
title: "Allowlist and denylist bypass on serializers, binders, and ORM attribute guards"
description: Filter evasion on bound field names with case variants, Unicode homoglyphs, duplicate keys, or nested paths the binder still maps.
keywords:
  - field allowlist
  - parameter pollution
  - nested key
---

# Allowlist bypass

## Context

A naive server strips `admin` from the top-level object but not `Admin`, a homoglyph, or `profile.admin`. Parsers that last-wins on duplicate keys or merge nested objects can still set a protected field. This is a binder and parser interaction problem, not a new BOLA primitive on the id.

## Theory

JSON duplicate key rules, `application/x-www-form-urlencoded` repeated keys, and `application/json` with `Content-Type` tricks can change which value the framework reads. Each stack documents its own rules; the offensive work is to try the same logical key in every encoding the stack accepts.

## Practice

### Resend the same logical field with case and nesting variants

- In a lab, rotate `role`, `Role`, `profile[role]`, and nested `{"profile":{"role":"admin"}}` against the same update route and diff the stored object.

## Tools

- **Burp Suite**
- **curl**
