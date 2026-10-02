---
title: "PHP disable_functions bypass: reaching command execution past the denylist"
description: "Getting OS command execution when PHP disable_functions blocks system/exec: unblocked alternatives, LD_PRELOAD via a spawned process, mail-based injection, and FFI."
keywords:
  - disable_functions bypass
  - LD_PRELOAD
  - putenv
  - FFI
  - mail
  - Chankro
---

# PHP disable functions bypass

`disable_functions` is a PHP denylist of function names. It is applied by the interpreter, it is a denylist rather than an allowlist, and it cannot cover every route to the OS, so from a PHP foothold (webshell, code-eval primitive) there are several ways to reach command execution anyway.

## Enumerate what remains

First list the actual restriction and look for an unblocked execution sink:

```php
echo ini_get('disable_functions');
```

Admins commonly block `system, exec, shell_exec, passthru, popen, proc_open` but miss one of: `proc_open`, `popen`, `pcntl_exec`, `mail`, `mb_send_mail`, `imap_open`, `dl`, `putenv`, `error_log`, or the FFI API. Any single survivor can be enough.

## LD_PRELOAD via a spawned subprocess

The strongest generic bypass: make PHP spawn *any* subprocess, and hijack it with a malicious shared object via `LD_PRELOAD`. PHP sets the environment with `putenv`, and functions like `mail()`/`mb_send_mail()` internally fork `/usr/sbin/sendmail`. Point `LD_PRELOAD` at your `.so`, trigger the subprocess, and your library's constructor runs the command outside PHP's restrictions:

```php
putenv("LD_PRELOAD=/tmp/x.so");
mail("a","a","a","a");          // forks sendmail, loads x.so
```

The shared object defines a function that runs on load and executes the command. **Chankro** automates building the `.so` and the PHP loader:

```bash
python3 chankro.py --arch 64 --input cmd.sh --output shell.php --path /var/www/html
```

This works even when every `*exec*` function is disabled, because the execution happens in the spawned C process, not in PHP.

## FFI

If FFI is usable at request time, call libc directly from PHP and bypass the denylist entirely. The precondition is `ffi.enable=true`, not merely the extension being loaded: the default `ffi.enable=preload` restricts `FFI::cdef()` to CLI and preloaded code and blocks it in a normal FPM/mod_php web request, so a webshell `FFI::cdef()` works only where an operator set `ffi.enable=true` (uncommon in production).

```php
$ffi = FFI::cdef("int system(const char *command);");
$ffi->system("id");
```

## Other routes

- **`imap_open`**: historically allowed command injection through its mailbox argument (`-oProxyCommand`), a one-call RCE when IMAP is available.
- **`dl()`**: load a crafted PHP extension if dynamic loading is permitted.
- **Known interpreter exploits**: version-specific UAF/bug chains (for example the PHP 7.x `GC`/`uaf` exploits) disable the restriction from within; match to the exact PHP version.

## Exploitation notes

- Confirm `open_basedir` separately; it can stop you writing the `.so` or reading your tools, in which case see [open_basedir bypass](php-open-basedir-bypass.md) first.
- You need a writable path for the `.so`/payload (uploads, `/tmp`, session dir).
- Prefer the LD_PRELOAD route for reliability across hardened hosts; FFI and `imap_open` are faster when available.

## Tools

- **Chankro** (LD_PRELOAD + sendmail), prebuilt `disable_functions` exploit shells matched to PHP versions.

## References

- PHP manual: disable_functions, FFI
- Chankro project
