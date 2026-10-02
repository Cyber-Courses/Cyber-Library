---
title: "PostgreSQL out-of-band SQLi: DNS and network exfiltration via SQL primitives"
description: Using COPY TO PROGRAM, extensions, or DNS side channels where the database can reach the network, highly environment-specific.
keywords:
  - OOB SQL injection
  - PostgreSQL
---

# Out of band

## Context

Maps to Library Structure **Out of Band** for PostgreSQL. **Pl/pgSQL** blocks that run **COPY … PROGRAM** with **nslookup**-style commands are a **research/lab** pattern; real app SQLi rarely chains to **superuser** **PROGRAM** rights.
