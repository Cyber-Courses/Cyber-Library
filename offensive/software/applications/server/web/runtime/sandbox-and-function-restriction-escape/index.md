---
title: "Sandbox and function restriction escape: defeating runtime containment"
order: 3
description: "Escaping runtime-level containment after code execution: PHP disable_functions and open_basedir bypasses, Python sandbox escapes, and Node.js vm escapes."
keywords:
  - disable_functions bypass
  - open_basedir bypass
  - python sandbox escape
  - node vm escape
  - runtime containment
---

# Sandbox and function restriction escape

When an attacker already has code execution inside a runtime but that runtime is **contained**, the next step is escaping the containment. These controls live at the runtime layer: PHP's `disable_functions` and `open_basedir`, Python's restricted-execution attempts, and Node's `vm`/`vm2` module isolation. All of them try to fence off a language that was not designed to fence itself, and each has well-worn escapes.

## When this applies

You reach this section after a foothold: an uploaded webshell that finds `system` disabled, an SSTI or eval primitive confined to a sandbox, a `vm`-wrapped user script. The question is no longer "can I run code" but "can I reach the OS and filesystem the runtime is trying to keep me from."

## The recurring moves

- **Find an unblocked equivalent.** A denylist (`disable_functions`) rarely covers every path to the same capability, so enumerate what remains.
- **Go under the language.** Load native code (LD_PRELOAD via a spawned subprocess, FFI, an extension) so the restriction, enforced in the interpreter, no longer applies.
- **Reconstruct the primitive.** In Python and Node sandboxes, walk the object graph back to builtins, `process`, or a class that bridges to the OS.

## Pages

- **[PHP disable functions bypass](php-disable-functions-bypass.md)**: reaching command execution when `disable_functions` blocks the obvious calls.
- **[PHP open_basedir bypass](php-open-basedir-bypass.md)**: reading and writing outside the permitted path tree.
- **[Python sandbox escape](python-sandbox-escape.md)**: recovering builtins and OS access from a restricted `eval`/`exec`.
- **[Node.js vm escape](nodejs-vm-escape.md)**: breaking out of the `vm`/`vm2` module to `process` and `require`.

## References

- PHP manual: disable_functions, open_basedir
- HackTricks: bypass PHP restrictions (reference index)
