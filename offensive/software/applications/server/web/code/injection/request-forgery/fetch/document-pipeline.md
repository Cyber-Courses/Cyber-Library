---
title: "Document and media pipelines in SSRF: HTML-to-PDF, image proxies, and server-side rendering fetches"
description: SSRF through converters that fetch user-supplied URLs to render PDFs, thumbnails, previews, or OG metadata.
keywords:
  - SSRF
  - HTML to PDF
  - image proxy
---

# Document pipeline SSRF

**Preview** and **export** features often pass a URL to **wkhtmltopdf**, **Chromium print**, **ImageMagick**, or a microservice that fetches first, renders second. That fetcher may follow redirects, allow `file://` in older builds, or leak response bytes in the generated file or error text.

## See also

- [Fetch client (parent)](index.md)
- [file:// scheme](../scheme/file.md)
