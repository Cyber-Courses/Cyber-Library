---
title: "IIS NTFS filename tricks: ::$DATA, trailing dots, and short-name enumeration"
description: "Abusing NTFS filename handling on IIS to disclose source and enumerate hidden files: alternate data streams (::$DATA), trailing dots and spaces, and 8.3 short-name differential enumeration."
keywords:
  - ::$DATA
  - NTFS ADS
  - 8.3 short name
  - trailing dot
  - IIS source disclosure
---

# NTFS filename tricks

NTFS accepts several filename forms that resolve to the same file but look different to a handler or filter. On IIS these disclose script source and enumerate files that are not linked anywhere.

## Alternate data streams (::$DATA)

Every NTFS file has a default data stream named `$DATA`. Appending `::$DATA` refers to the same bytes but changes the extension the handler sees, so a script handler may not fire and IIS returns the **raw source**:

```
GET /default.aspx::$DATA HTTP/1.1      # ASPX source, not executed
GET /web.config::$DATA HTTP/1.1
GET /login.asp::$DATA HTTP/1.1
```

A related stream form is `:$I30:$INDEX_ALLOCATION` appended to a directory name.

## Trailing dots and spaces

Windows strips trailing dots and spaces when opening a file, but a handler or filter may key on the literal request. `app.aspx.` (trailing dot) or `app.aspx%20` opens `app.aspx` on disk while presenting a different suffix to the handler, yielding source disclosure or a handler bypass:

```
GET /secret.aspx. HTTP/1.1
GET /secret.aspx%20 HTTP/1.1
```

## 8.3 short-name enumeration

NTFS keeps legacy 8.3 short names (`PROGRA~1`). IIS reveals their existence through differential responses to a tilde pattern, letting an attacker recover otherwise-unknown filenames a character at a time:

```
GET /secr~1.asp*~ HTTP/1.1     # tilde-probe pattern used by short-name scanners
```

A short-name scanner automates this to reconstruct partial names of config, backup, and admin files that are not linked, which you then fetch in full (and via `::$DATA` for source).

## Exploitation

- Append `::$DATA` and trailing `.`/`%20` to known executable scripts (`.aspx`, `.asp`, `.ashx`, `.asmx`, `.config`) to pull source and secrets.
- Run an IIS short-name scanner to enumerate hidden filenames, then request the full names and their `::$DATA` source.
- Send with `curl --path-as-is` / Burp so the exact bytes reach IIS.

## Tools

- **IIS-ShortName-Scanner**; **curl --path-as-is** / Burp.

## References

- Microsoft: NTFS alternate data streams, 8.3 naming
- Soroush Dalili: IIS short-name and ::$DATA research
