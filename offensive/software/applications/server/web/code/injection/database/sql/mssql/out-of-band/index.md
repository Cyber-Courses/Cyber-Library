---
title: "MSSQL out-of-band SQL injection: DNS and UNC-style network callbacks"
description: Out-of-band channels in SQL Server—DNS labels and UNC paths that can trigger network activity when injection can influence string arguments to privileged functions.
keywords:
  - DNS exfiltration
  - UNC path
  - MSSQL OOB
---
# Out of band (MSSQL)

When **inline query results** are hard to read, attackers may coerce **outbound network** behavior: **DNS** subdomains embedding data, or **UNC** paths that cause the server to **resolve** or **authenticate** to a listener. Prerequisites include **string-building** in an injectable context and **network egress** (or SMB listener reachability).

| Topic | Path |
|-------|------|
| DNS exfiltration | [DNS exfiltration](dns-exfiltration.md) |
| UNC path | [UNC path](unc-path.md) |

## Tradecraft overview

Map the injection class (reflection in page, errors, timing), then pick the branch that matches what the stack leaks: **union** for visible rows, **blind**/**time** for suppressed output, **stacked**/**exec** when the driver allows multiple batches. Escalate to file, OOB, or OS only after you know the effective database user and enabled features.

## Scope

Use only in **authorized** penetration tests, red-team engagements, CTFs, and isolated lab systems.

## See also

- [MSSQL (SQLi)](../index.md)
