---
title: "Perl server-side template injection"
description: "SSTI in Perl's Template Toolkit: the PERL and RAWPERL blocks gated by EVAL_PERL, and the system/exec plugins that reach command execution."
keywords:
  - Perl SSTI
  - Template Toolkit
  - EVAL_PERL
  - PERL block
---

# Perl

Perl's dominant engine is Template Toolkit (TT2), which uses `[% ... %]` directives. TT2 was designed to separate logic from presentation, so running raw Perl is gated behind an explicit option rather than available by default. Exploitation therefore splits into two cases: the `PERL`/`RAWPERL` blocks when `EVAL_PERL` is enabled, and the plugin and filter mechanism (notably the `system` and `exec` directives and the `Datafile`/`redirect` features) which can reach the OS or the filesystem depending on configuration.

Confirm with `[% 7*7 %]` rendering `49`.

## Engines

- **[Template Toolkit](template-toolkit.md)**: `[% PERL %]` blocks under EVAL_PERL, the `system`/`exec` filters, and file-write via `redirect`.

## References

- Template Toolkit documentation (template2)
- PortSwigger Web Security Academy: Server-side template injection
