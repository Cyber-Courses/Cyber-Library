---
title: "Oracle out-of-band SQL injection: UTL_HTTP, UTL_INADDR, and XML callbacks"
description: Out-of-band channels in Oracle—HTTP requests, DNS-style host resolution, and XML-driven fetches when packages are granted.
keywords:
  - UTL_HTTP
  - UTL_INADDR
  - EXTRACTVALUE
---
# Out of band (Oracle)

| Topic | Path |
|-------|------|
| EXTRACTVALUE | [EXTRACTVALUE](extractvalue.md) |
| UTL HTTP | [UTL HTTP](utl-http.md) |
| UTL INADDR | [UTL INADDR](utl-inaddr.md) |

## Tradecraft overview

Map the injection class (reflection in page, errors, timing), then pick the branch that matches what the stack leaks: **union** for visible rows, **blind**/**time** for suppressed output, **stacked**/**exec** when the driver allows multiple batches. Escalate to file, OOB, or OS only after you know the effective database user and enabled features.

## Scope

Use only in **authorized** penetration tests, red-team engagements, CTFs, and isolated lab systems.

## See also

- [Oracle Database (SQLi)](../index.md)
