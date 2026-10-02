---
title: "data://, php://input, and expect:// wrappers: direct inclusion to RCE"
description: "Supplying attacker-controlled content straight to a PHP include through the data:// and php://input wrappers, and command execution via expect://, when remote URL inclusion is disabled."
keywords:
  - data wrapper
  - php://input
  - expect://
  - RFI
  - allow_url_include
---

# data and input wrappers

When an `include`/`require` sink takes attacker-influenced input, several wrappers let you supply the *content to execute* directly, without hosting a remote file. These are the go-to escalation from LFI to RCE when classic remote file inclusion is blocked but the relevant wrapper is enabled.

## data://

The `data://` wrapper inlines content in the path itself. Pointed at an `include`, it executes the inlined PHP:

```
data://text/plain,<?php system($_GET['c']); ?>
data://text/plain;base64,PD9waHAgc3lzdGVtKCRfR0VUWydjJ10pOyA/Pg==
```

`data://` requires `allow_url_include=On`. Base64 form avoids awkward characters in the URL.

## php://input

`php://input` is the raw request body. When the sink includes it, put your PHP in the POST body:

```
POST /?page=php://input HTTP/1.1
Content-Type: application/x-www-form-urlencoded

<?php system($_GET['c']); ?>
```

This also needs `allow_url_include=On`, but it keeps the payload out of the URL and logs, which can matter for stealth and length limits.

## expect://

The `expect://` wrapper runs a command through the Expect extension, so inclusion is direct command execution:

```
expect://id
```

This requires the (uncommon) `expect` PECL extension to be loaded, so it is a situational win rather than a first choice.

## Choosing a vector

- If `allow_url_include=On`: `data://` or `php://input` give immediate RCE with the simplest payloads.
- If `allow_url_include=Off`: these are unavailable; pivot to [filter chains to RCE](filter-chains-to-rce.md) (no config requirement) or [phar deserialization](phar-deserialization.md).
- `expect://` only when the extension is present.

## Exploitation notes

- Confirm the sink truly executes (an `include`/`require`), not merely reads; a read sink returns your payload as text instead of running it.
- Watch for appended extensions or path prefixes the app adds, and for input filters on `php://`, `data:`, or `<?php`, which you can often dodge with casing, base64 (`data://...;base64,`), or `<?=` short tags.
- Combine with [php://filter](php-filter-wrapper.md) source disclosure first to learn exactly how the sink builds the path.

## Tools

- **curl** / Burp Repeater.

## References

- PHP manual: data://, php://input, supported protocols
- PortSwigger Web Security Academy: LFI to RCE
