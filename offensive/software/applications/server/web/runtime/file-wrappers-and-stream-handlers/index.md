---
title: "File wrappers and stream handlers: escalating file operations through engine pseudo-protocols"
order: 2
description: "How runtime stream wrappers (mostly PHP's php://, phar://, data://) turn a file read or include into source disclosure, deserialization, and remote code execution."
keywords:
  - php wrappers
  - php://filter
  - phar
  - data wrapper
  - stream handler
  - LFI to RCE
---

# File wrappers and stream handlers

Language runtimes expose **stream wrappers**: pseudo-protocols that let ordinary file functions operate on things that are not plain files. PHP is by far the richest target, with `php://`, `phar://`, `data://`, `zip://`, and more. When an application passes attacker-influenced input to a file function (`include`, `fopen`, `file_get_contents`, `getimagesize`, `copy`), these wrappers escalate what looks like a file read into source disclosure, deserialization, or code execution. This is a runtime-layer issue because the capability comes from the engine, not from the application's own logic.

## Why it matters

A path-handling bug that would otherwise be a limited local file read becomes much more when wrappers are available:

- `php://filter` reads and transforms file contents, dumping source code, and chained filters can synthesize executable PHP from no file at all.
- `phar://` deserializes archive metadata when any file function touches the path, reaching object injection without an `unserialize()` call.
- `data://` and `php://input` supply attacker-controlled content directly to an `include` (both require `allow_url_include=On`); when that setting is off, `php://filter` chains still reach code execution with no config dependency.

## Pages

- **[php filter wrapper](php-filter-wrapper.md)**: reading source and arbitrary files with `php://filter` and conversion filters.
- **[Filter chains to RCE](filter-chains-to-rce.md)**: synthesizing a PHP payload purely through chained conversion filters.
- **[Phar deserialization](phar-deserialization.md)**: triggering object injection via `phar://` on file operations.
- **[data and input wrappers](data-and-input-wrappers.md)**: `data://`, `php://input`, and `expect://` for direct inclusion and execution.

## References

- PHP manual: supported protocols and wrappers
- PortSwigger Web Security Academy: File path traversal, LFI
