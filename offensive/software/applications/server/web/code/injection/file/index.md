---
title: "File and path injection in web applications: traversal, upload, and dynamic inclusion"
description: How untrusted input steers which file a server reads, writes, or executes, directory escape, attacker-controlled uploads, and include/require driven by request parameters, and how each reaches file disclosure and code execution.
keywords:
  - path traversal
  - file upload
  - local file inclusion
  - LFI
  - RFI
  - directory traversal
  - web shell
---

# File inclusion and path handling

**File and path injection** is the family of vulnerabilities in which application code builds a filesystem path, or chooses a file to read, write, include, or extract, from untrusted input, and the attacker steers that operation to a target the developer never intended. The payoff ranges from **reading sensitive files** (source, configuration, credentials, keys) to **writing executable content** into a served directory and reaching **remote code execution (RCE)** in the context of the application's service account.

## Overview

Web applications touch the filesystem constantly: they serve a user's export, store an uploaded avatar, render a template chosen by a route, extract an archive, or `include` a module named by a parameter. What makes these exploitable is the same root cause each time, **a filename or path crosses from data into a resolution that the attacker partially controls**. Three distinct mechanisms produce three distinct bug classes, and keeping them separate clarifies the attack surface:

1. **Path traversal.** A parameter that selects a file is concatenated into a path that resolves relative to a weak root. Sequences like `../`, their encodings, or absolute paths escape the intended directory and reach arbitrary files for read or write. **Zip slip** is the archive-extraction variant, where traversal lives inside stored member names. See [Path traversal](path-traversal.md).

2. **Unrestricted file upload.** The application accepts a file whose **name, extension, content type, or storage location** is attacker-influenced. Weak type checks and a web-served destination turn an upload into a stored **web shell**, stored XSS, or an SSRF/XXE follow-on. See [Unrestricted file upload](unrestricted-file-upload.md).

3. **Dynamic file inclusion.** A request parameter is mapped to a path or module identifier for `include`/`require`, a JSP path, or a template name. **Local file inclusion (LFI)** reads, and sometimes executes, files already on disk; **remote file inclusion (RFI)** pulls attacker-hosted code on stacks that permit it. See [Dynamic file inclusion](dynamic-file-inclusion.md).

The decisive questions when assessing a sink are always: *does my input choose the file, the directory, or both; is the result read, written, or executed; and what normalization happens between the request and the filesystem call?*

## Why it reaches the filesystem

- **String concatenation feels natural** for building a path, `base + "/" + userValue`, and a value that looks like a plain filename can carry `..`, a null byte, or a leading `/`.
- **Normalization order is subtle.** URL-decoding, path-joining, and canonicalization happen at different layers; a check performed before the final decode is routinely bypassed.
- **Type checks inspect the wrong signal.** An extension, a `Content-Type` header, or a magic-byte sniff each describes something the attacker controls, and they frequently disagree with how the server later *serves* the file.
- **Second-order flows.** A filename stored earlier is later joined into a path or passed to `include` far from where it entered, so the taint is easy to miss.

## Impact

File read alone is often decisive, source code, `/etc/passwd`, cloud metadata, framework secrets, and session files all live on disk. File *write* into a web root yields code execution directly. Inclusion sinks bridge the two: an LFI that reaches a log, a session file, or a PHP wrapper becomes RCE. As with command injection, the ceiling is set by the **service account's privileges** and the **network position** of the host, a worker that can read internal configuration or reach a metadata endpoint is frequently worth more than a shell on an isolated box. Chains often terminate in **[Command injection](../command/index.md)** when the final step hands a path to a shell.

## Pages

| Page | Focus |
|------|--------|
| [Path traversal](path-traversal.md) | Directory escape via `../`, encodings, absolute paths, and zip slip for read and write |
| [Unrestricted file upload](unrestricted-file-upload.md) | Type, extension, magic-byte, and polyglot bypasses; path to web-shell execution |
| [Dynamic file inclusion](dynamic-file-inclusion.md) | LFI and RFI in includes and templates; wrappers, log poisoning, session files, and the path to RCE |

## References

- [OWASP: Path Traversal](https://owasp.org/www-community/attacks/Path_Traversal)
- [OWASP: Unrestricted File Upload](https://owasp.org/www-community/vulnerabilities/Unrestricted_File_Upload)
- [CWE-22: Improper Limitation of a Pathname to a Restricted Directory (Path Traversal)](https://cwe.mitre.org/data/definitions/22.html)
- [CWE-98: Improper Control of Filename for Include/Require Statement (File Inclusion)](https://cwe.mitre.org/data/definitions/98.html)
- [CWE-434: Unrestricted Upload of File with Dangerous Type](https://cwe.mitre.org/data/definitions/434.html)
- [PayloadsAllTheThings: File Inclusion and Directory Traversal](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/File%20Inclusion)
