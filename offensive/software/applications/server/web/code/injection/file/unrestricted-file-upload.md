---
title: "Unrestricted file upload: type-check bypasses and the path to web-shell execution"
description: Exploiting multipart uploads where extension, content type, magic bytes, or storage location are attacker-controlled—extension tricks, MIME spoofing, magic-byte and polyglot bypasses—leading to stored web shells, XSS, and SSRF/XXE follow-ons.
keywords:
  - file upload
  - web shell
  - unrestricted upload
  - MIME sniffing
  - polyglot
  - double extension
  - RCE
---

# Unrestricted file upload

**Unrestricted file upload** exploits code that accepts a user-supplied file whose **name, extension, content type, content, or storage location** is attacker-influenced, and later stores or serves it in a way that grants the attacker capability. The headline outcome is a **stored web shell**—an executable file written into a served directory—yielding remote code execution as the service account. Weaker configurations still give stored XSS, client-side code execution, or a server-side parsing follow-on (SSRF, XXE, decompression abuse).

> **Scope.** For authorized penetration tests, red-team engagements, CTF labs, and code review of systems you own or are contracted to assess. Use only against systems you are permitted to test.

## Overview

An upload is dangerous when three independent facts line up: the server **accepts** the file, **stores** it somewhere reachable, and later **serves or interprets** it as something active. A filter that inspects only one signal—say, the declared `Content-Type`—leaves the others free:

```
POST /upload  (multipart/form-data)
filename="avatar.php"
Content-Type: image/png        <-- attacker-set header, trusted by a naive check
<?php system($_GET['c']); ?>
```

If the handler trusts the header, writes to `/var/www/html/uploads/avatar.php`, and the web server maps `.php` there to the PHP interpreter, requesting the stored URL executes the payload. The attacker never bypassed the filesystem—they defeated a **type decision** that looked at a value they controlled.

## The signals a filter checks (and how each fails)

| Signal | What it is | Why it is bypassable |
|--------|-----------|----------------------|
| Extension (blocklist) | Reject `.php`, `.jsp`, … | Incomplete list: `.php5`, `.phtml`, `.phar`, `.pht`, `.asp;.jpg`, `.jspx` |
| Extension (allowlist) | Permit only `.jpg`, `.png` | Double extension `shell.php.jpg`, trailing dot/space, null byte `shell.php%00.jpg` |
| `Content-Type` header | Client-declared MIME | Attacker-controlled; set to `image/png` freely |
| Magic bytes / sniff | Server reads file header | Prepend a valid signature (`GIF89a;`) before the payload |
| Size / dimensions | Image must decode | A polyglot both decodes as an image and executes as code |

Because these checks are independent, the reliable approach is to satisfy each one *simultaneously* with a single crafted file.

## Bypass techniques

### Extension handling

- **Double extension:** `shell.php.jpg` passes a check that only inspects the final segment, yet executes on a server that routes any file *containing* `.php`, or when combined with an Apache `AddHandler` misconfiguration.
- **Alternate executable extensions:** `.phtml`, `.php3/4/5/7`, `.phar`, `.pht` (PHP); `.jspx`, `.jspf` (Java); `.asmx`, `.ashx`, `.cshtml` (.NET). A blocklist rarely enumerates all of them.
- **Case and trailing characters:** `shell.pHp`, `shell.php.` , `shell.php%20`, `shell.php::$DATA` (NTFS alternate data stream) defeat exact-match comparisons.
- **Null-byte truncation:** `shell.php%00.jpg` on legacy stacks truncates at the null, storing `shell.php`.

### Content-type and magic-byte spoofing

The multipart `Content-Type` is set by the client, so a server that trusts it is trivially defeated. When the server sniffs **magic bytes**, prepend a legitimate signature and let the interpreter ignore the leading bytes:

```
GIF89a;
<?php system($_GET['c']); ?>
```

PHP ignores content before `<?php`, so the file is a valid GIF *and* executable. The same prefix trick works for `%PDF-`, `BM` (bitmap), and others.

### Polyglots

A **polyglot** is a single file valid under two formats at once—commonly a real image that also carries script. A JPEG with a PHP payload in a comment segment (`COM` marker) survives magic-byte checks and even some re-encoders, while still executing if the server runs the file. Polyglots also enable stored XSS where the file is served inline: an "image" that a browser renders as HTML (content-type sniffing) runs the embedded script.

### Server-side parsing follow-ons

Even when execution is blocked, *the act of processing the upload* is attack surface:

- **SVG upload → XSS/XXE/SSRF.** SVG is XML: embedded `<script>` yields stored XSS when served inline; an external entity (`<!ENTITY ... SYSTEM "file:///etc/passwd">`) yields XXE or SSRF when the server parses it.
- **Image library abuse.** Formats handled by ImageMagick/GhostScript have historically allowed command execution or file read during decode (e.g., crafted postscript/MVG).
- **Archive uploads.** A zip/tar processed server-side invites [zip slip](path-traversal.md) and decompression-bomb denial of service.
- **XXE via office/OOXML** files that are parsed as XML.

## Reaching execution

Writing the file is only half the exploit; it must be **served from a location where the runtime interprets it**. Three conditions to establish:

1. **Storage path.** Is the upload written under the web root, or to an object store / path that is proxied back? Combine with [path traversal](path-traversal.md) in the filename (`../../var/www/html/x.php`) when the directory is otherwise outside the served tree.
2. **Serving path.** Request the file both through the application (which may re-serve with a safe content-type) and through the **static mapping** (direct URL). Direct access often bypasses an app-layer content-type override.
3. **Handler mapping.** Does the server map your extension to an interpreter in that directory? A `.php` under a path with `php_admin_flag engine off` will not run; the same file elsewhere will. `.htaccess` upload (where permitted) can itself add a handler that turns an innocuous extension into executable code.

Once a shell lands and runs, the engagement typically pivots off the upload point to a reverse shell or an interactive foothold, exactly as with [command injection](../command/index.md).

## Exploitation workflow

1. **Enumerate the upload surface:** avatars, document imports, attachments, import/restore, profile media, signature images.
2. **Baseline the filter:** upload a clean allowed file, note the returned path, content-type, and whether the name is preserved or randomized.
3. **Probe one signal at a time:** flip the `Content-Type`, then the extension, then prepend magic bytes, to learn which check is enforced.
4. **Combine into one crafted file** satisfying every observed check, with an executable payload.
5. **Locate the served copy** via the app and via the static URL, then trigger it.

## Tools

- **[Burp Suite](https://portswigger.net/burp)** (Repeater, Intruder) for manipulating multipart fields, extensions, and content-type headers.
- **[ExifTool](https://exiftool.org/)** to embed payloads in image metadata/comment segments for polyglots.
- **[fuxploider](https://github.com/almandin/fuxploider)** to automate upload-filter detection and extension fuzzing.
- **[Upload_Bypass](https://github.com/sAjibuu/Upload_Bypass)** for systematic extension and content-type bypass generation.

## References

- [OWASP: Unrestricted File Upload](https://owasp.org/www-community/vulnerabilities/Unrestricted_File_Upload)
- [PortSwigger Web Security Academy: File upload vulnerabilities](https://portswigger.net/web-security/file-upload)
- [CWE-434: Unrestricted Upload of File with Dangerous Type](https://cwe.mitre.org/data/definitions/434.html)
- [PayloadsAllTheThings: Upload Insecure Files](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Upload%20Insecure%20Files)
