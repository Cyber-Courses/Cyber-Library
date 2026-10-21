---
title: "IIS NTFS filename tricks: ::$DATA, trailing dots, and short-name enumeration"
order: 1
description: "Testing version- and configuration-dependent IIS filename behavior involving NTFS alternate streams, trailing characters, and 8.3 short-name enumeration."
keywords:
  - ::$DATA
  - NTFS ADS
  - 8.3 short name
  - trailing dot
  - IIS source disclosure
---

# NTFS filename tricks

NTFS accepts filename forms that may resolve to the same file while looking different to a handler or filter. Historical IIS versions and particular handler configurations have exposed source or produced distinguishable responses for some of these forms. The filesystem syntax alone does not prove an IIS vulnerability: HTTP.sys, request filtering, handler mapping, the application framework, and NTFS volume settings all affect the result.

## Alternate data streams (::$DATA)

Every NTFS file has an unnamed default data stream of type `$DATA`. Historical handler-confusion issues used an appended stream name to make a request look different from the script's ordinary extension:

```
GET /default.aspx::$DATA HTTP/1.1
GET /web.config::$DATA HTTP/1.1
GET /login.asp::$DATA HTTP/1.1
```

Do not assume these requests return source on a current server. They may be rejected, normalized, mapped to the original handler, or answered as not found. Source disclosure requires a specific IIS version and handler path that opens the stream while bypassing script execution. A related historical probe appends `:$I30:$INDEX_ALLOCATION` to a directory name.

## Trailing dots and spaces

Win32 filename normalization can remove trailing dots or spaces, while web-layer parsing may reject or transform them earlier. Historical mismatches used `app.aspx.` or `app.aspx%20` to test whether a filter and handler interpreted the path differently:

```
GET /secret.aspx. HTTP/1.1
GET /secret.aspx%20 HTTP/1.1
```

## 8.3 short-name enumeration

NTFS can maintain legacy 8.3 short names (`PROGRA~1`). Where 8.3 name creation is enabled and the IIS request path produces distinguishable responses, tilde probes can reveal whether a matching short name exists:

```
GET /secr~1.asp*~ HTTP/1.1     # tilde-probe pattern used by short-name scanners
```

A short-name scanner automates differential probes to reconstruct partial names. Enumeration does not guarantee that the long filename can be recovered uniquely, fetched without authorization, or disclosed through an alternate stream.

## Exploitation

- Compare the status, body length, headers, and timing of ordinary and alternate-name requests; a single error response is not evidence of source disclosure.
- Confirm the IIS and Windows versions, request-filtering configuration, handler mapping, and NTFS 8.3 setting before assigning impact.
- Run a short-name scanner only after a manual positive and negative tilde probe establish a stable response difference.
- Send with `curl --path-as-is` / Burp so the exact bytes reach IIS.

## Tools

- **IIS-ShortName-Scanner**; **curl --path-as-is** / Burp.

## References

- [Microsoft: file streams](https://learn.microsoft.com/en-us/windows/win32/fileio/file-streams)
- [Microsoft IIS Support Blog: IIS short-name enumeration](https://techcommunity.microsoft.com/blog/iis-support-blog/iis-short-name-enumeration/3951320)
