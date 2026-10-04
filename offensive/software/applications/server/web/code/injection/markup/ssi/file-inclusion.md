---
title: "SSI file inclusion"
description: "The include directive pulls files and server-side resources into an SSI-parsed page; file and virtual paths plus traversal let an attacker read local files and reach internal endpoints."
keywords:
  - SSI include file
  - include virtual
  - SSI path traversal
  - server-side resource inclusion
  - local file read
---

# File inclusion

The `include` directive inserts the contents of another resource into the page at parse time. When attacker input reaches an SSI-parsed document, an injected `include` reads files from disk or pulls in server-side resources, and path traversal in the parameter widens that reach to files outside the document root.

## The two forms

`include` takes one of two parameters, and the difference decides what the server resolves.

```
<!--#include file="footer.html"-->
<!--#include virtual="/includes/footer.html"-->
```

`file` names a path relative to the current document and, by default, does not permit `..` to climb above it, though misconfiguration and parser quirks frequently loosen that. `virtual` takes a URL-space path relative to the document root and is resolved by the server, so it can reach anything the web server maps, including dynamic handlers and CGI output.

## Reading local files

`file` is the route to the filesystem, but on the standard `mod_include` it resolves relative to the current document and rejects an absolute path, so reaching a system file means climbing out with traversal rather than naming it directly:

```
<!--#include file="../../../../../../etc/passwd"-->
```

On Windows targets, adjust separators and targets:

```
<!--#include file="..\..\..\..\..\..\windows\win.ini"-->
```

Encoded traversal can slip past input filters that only match literal `../`:

```
<!--#include file="..%2f..%2f..%2f..%2f..%2f..%2fetc/passwd"-->
```

`virtual` resolves in URL space, not on disk, so it reads an OS path like `/etc/passwd` only where an explicit alias or mapping exposes it; its real reach is the resources the server maps, covered next.

## Including server-side resources

Because `virtual` is resolved in URL space, it reaches dynamic endpoints rather than only static files. Including a server-side script pulls that endpoint's rendered output into the response, which can expose internal-only pages or trigger their side effects in the context of the server:

```
<!--#include virtual="/admin/status"-->
<!--#include virtual="/cgi-bin/internal.cgi?debug=1"-->
<!--#include virtual="/server-status"-->
```

An `include virtual` is an internal subrequest, not a fresh HTTP connection from localhost, and it keeps the original request and client context for access checks, so it does not forge a loopback origin or bypass a source-address restriction. What it gains is reaching resources in URL space that are not linked in normal navigation, pulling their rendered output into the response, and triggering their side effects during page assembly. Where the included handler reflects its own parameters, the inclusion can also become a pivot to further injection.

## Confirming and iterating

Start with a known-present include to confirm the directive is parsed, then move to targets of interest:

```
<!--#include virtual="/robots.txt"-->
```

A response that now contains the `robots.txt` body confirms resolution. Use the on-disk path disclosed by `echo var="SCRIPT_FILENAME"` or `PATH_TRANSLATED` to compute the exact number of traversal steps needed, which removes the guesswork from stacking `../` sequences.

If `include` returns an error rather than content (for example `[an error occurred while processing this directive]`), the directive was parsed but the path failed; adjust the path or switch between `file` and `virtual`.

## Tools

- **Burp Suite**: injecting `include` directives and iterating traversal depth with Intruder.
- Manual testing with crafted `<!--#include file-->` and `<!--#include virtual-->` payloads.

## References

- [OWASP: Server-Side Includes (SSI) Injection](https://owasp.org/www-community/attacks/Server-Side_Includes_(SSI)_Injection)
- [PayloadsAllTheThings: Server Side Inclusion / Edge Side Inclusion Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Server%20Side%20Include%20Injection)
