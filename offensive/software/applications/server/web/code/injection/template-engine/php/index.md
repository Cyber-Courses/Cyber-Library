---
title: "PHP server-side template injection"
order: 4
description: "SSTI in PHP template engines: Twig (Symfony) filter-callback RCE and Smarty's PHP execution constructs."
keywords:
  - PHP SSTI
  - Twig
  - Smarty
  - Symfony template injection
---

# PHP

PHP's two common template engines both lead to code execution from a template injection, though by different routes. Twig (the default in Symfony and widely used standalone) is reached through its filter and function callbacks; Smarty exposes PHP more directly through dedicated constructs that have tightened over versions.

Detect Twig with `{{7*7}}` returning `49`, and Smarty with `{$smarty.version}` echoing the version string (Smarty uses single braces, so `{7*7}` is not a reliable probe and may error).

## Engines

- **[Twig](twig.md)**: `filter`/`map` callbacks to `system`, and the historical `_self` registerUndefinedFilterCallback.
- **[Smarty](smarty.md)**: the `{php}` tag and static-method constructs, with their version gating.

## References

- Twig and Smarty documentation (sandbox, security policy)
- PortSwigger Web Security Academy: Server-side template injection
