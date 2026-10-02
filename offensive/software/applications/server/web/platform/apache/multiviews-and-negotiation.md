---
title: "Apache MultiViews and content negotiation: file enumeration and source disclosure"
description: "Abusing Apache mod_negotiation MultiViews to enumerate files by base name, discover unreferenced variants, and disclose source or backups the server offers as negotiated representations."
keywords:
  - MultiViews
  - mod_negotiation
  - content negotiation
  - file enumeration
  - apache
---

# MultiViews and content negotiation

`mod_negotiation` with `Options +MultiViews` lets Apache serve the best matching variant when a request has no exact file. Requesting `/page` makes Apache look for `page.html`, `page.php`, `page.en`, `page.bak`, and so on, and pick one. For an attacker this turns a base name into a file-enumeration and source-disclosure oracle.

## Enumerating variants

Because Apache reports the available representations, a request for a base name (or a bad `Accept`) can reveal every variant that exists:

```
GET /index HTTP/1.1
Accept: application/x-does-not-exist
```

When no acceptable variant matches, `mod_negotiation` can return a **406 Not Acceptable** listing the available representations by filename, disclosing variants such as `index.php`, `index.php.bak`, `index.old`, and language-tagged copies that are not linked anywhere. This enumerates files without a wordlist.

## Disclosure and unreferenced content

- A negotiated `.bak`/`.old`/`.txt` variant is served with a static content type, disclosing source (overlaps [backup and temporary files](../general/backup-and-temporary-files.md), but here Apache volunteers the name).
- Base-name requests reach variants the application never links, exposing dev/test pages and older copies.
- `MultiViews` combined with a handler mapping can also change which variant executes versus discloses.

## Exploitation

- Request known page base names without extensions and with hostile `Accept` headers; read the 406 representation list for variant filenames.
- Pull each disclosed variant directly (the `.bak`/`.old` copies are the high-value ones for source and secrets).
- Feed discovered scripts back into the [handler and type mapping](handler-and-type-mapping.md) checks.

## Tools

- **curl**/Burp (manipulate `Accept`); content-discovery tooling for base names.

## References

- Apache httpd: mod_negotiation, Options MultiViews
- OWASP WSTG: Review old/unreferenced files
