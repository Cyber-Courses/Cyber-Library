---
title: "PHP open_basedir bypass: reading and writing outside the permitted tree"
description: "Escaping PHP's open_basedir path confinement to access the wider filesystem, through directory traversal quirks, symlinks, and subprocess or FFI routes that the interpreter check does not cover."
keywords:
  - open_basedir bypass
  - path confinement
  - symlink
  - chdir
  - FFI
---

# PHP open_basedir bypass

`open_basedir` confines PHP file operations to a configured path tree. Like `disable_functions`, it is enforced inside the interpreter's file API, so anything that reaches the filesystem *outside* that API, or that confuses the path check, escapes it. This matters when a foothold needs to read config/secrets or drop a payload beyond the allowed directory.

## In-interpreter tricks

Some bypasses stay within PHP by abusing how the check resolves paths:

- **`chdir()` plus relative traversal**: walking in and out of permitted directories with sequences of `chdir()` and `..` has historically desynchronized the resolved base from the checked base on some versions, allowing access outside the tree.
- **Symlinks**: if you can create a symlink inside an allowed directory pointing outside it (via an unzip, upload, or a reachable shell), following it through PHP file functions reads the target.
- **`glob://` and wrapper quirks**: enumeration wrappers sometimes list entries the direct check would deny.

These are version-sensitive; test against the exact PHP build.

## Going outside the file API

The robust escapes leave the interpreter's file layer entirely:

- **Subprocess**: if any command-execution path is available (see [disable_functions bypass](php-disable-functions-bypass.md)), run `cat`/`cp` as a child process; `open_basedir` does not apply to the spawned process.
- **FFI**: call libc `fopen`/`open`/`read` directly through `FFI::cdef`, bypassing PHP's checked file functions.
- **Loadable native code**: an `LD_PRELOAD` library or loaded extension reads and writes the real filesystem.

## Exploitation notes

- Check the current confinement and your position first:

  ```php
  echo ini_get('open_basedir');
  echo getcwd();
  ```

- `open_basedir` and `disable_functions` are independent; a host may set one and not the other. If command execution is available, the subprocess route solves both at once.
- The in-interpreter traversal tricks are the only option when no execution and no FFI exist, so keep them in reserve for tightly locked PHP-only footholds.

## Tools

- Version-matched `open_basedir` bypass snippets; FFI one-liners.

## References

- PHP manual: open_basedir, FFI
- Public open_basedir bypass writeups (version-specific)
