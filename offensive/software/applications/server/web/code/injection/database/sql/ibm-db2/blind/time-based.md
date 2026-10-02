---
title: "IBM Db2 time-based blind SQL injection: delays and heavy operations"
description: Timing inference in Db2—WAIT FOR, benchmark-style constructs, and conditional delays where permitted.
keywords:
  - WAIT FOR
  - time-based SQLi
  - Db2
---
# Time-based (IBM Db2)

Db2 supports **deliberate waits** in some contexts (for example **`WAIT FOR`** in procedures) and may use **expensive** **predicates** as weaker timing signals—exact primitives vary by **edition** and **configuration**.

## Context

Time-based SQLi uses deliberate delays tied to a boolean so you can measure **slow** vs **fast** responses when content is identical.
## Technique

Wrap a **sleep/wait** primitive in a conditional so only one branch pays the delay. Average multiple samples to beat jitter.
## Practice

- Baseline network RTT; use enough delay to clear noise on WAN targets.
- Combine with blind extraction primitives for data recovery.

## Tools

- **Burp Suite** (Repeater, Intruder)
- **sqlmap**
- Database client (SSMS, `sqlplus`, `sqlite3`, `db2`) for follow-up

## Scope

Use only in **authorized** penetration tests, red-team engagements, CTFs, and isolated lab systems.

## See also

- [Blind (IBM Db2)](index.md)
