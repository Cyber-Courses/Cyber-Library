---
title: "PostgreSQL error-based SQLi using XML functions: query_to_xml and database_to_xml"
description: Large XML aggregates that error or truncate in ways that leak data through exception messages.
keywords:
  - PostgreSQL XML
  - error-based SQLi
---

# XML helpers

## Context

**query_to_xml**, **database_to_xml**, **xmlagg** can produce **huge** or **invalid** XML that surfaces in errors. Use only on **lab** schemas; some payloads risk **DoS** from memory pressure—aligns with Library Structure **XML Helpers**.

## See also

- [Error-based (parent)](index.md)
