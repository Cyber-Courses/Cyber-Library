---
title: "Conditional time delays in MySQL time-based blind SQL injection (IF and benchmarks)"
description: Wrapping SLEEP in IF or CASE so only the true branch delays the response, leaking one bit per request.
keywords:
  - SLEEP SQL injection
  - time-based SQLi
  - MySQL
---

# Conditional delays

## Context

`AND IF(predicate, SLEEP(5), 0)` in an injectable **boolean** **context** makes **true** **predicates** **pause** **five** **seconds** (adjust for **WAF** **timeouts**). **Authorized** **labs** **only**.

## Theory

**Stack** **must** **allow** **stacked** **queries** **or** **you** **need** **a** **single** **expression** **slot** **that** **permits** **comma**-**separated** **function** **calls**. **Alternative**: **`UNION SELECT ...`** **with** **SLEEP** **in** **a** **column** **if** **union** **works** **but** **no** **visible** **output** **(rare)**.

## Practice

### Calibrate baseline latency

- Record **p50** **RTT** **without** **injection**. **Run** **IF(1=1,SLEEP(2),0)** **and** **IF(1=2,SLEEP(2),0)** **twenty** **times** **each**; **separate** **distributions** **should** **not** **overlap** **if** **the** **oracle** **works**.

### Bit extraction

- Replace `1=1` with **ASCII** **substring** **tests**; **use** **short** **SLEEP** **in** **staging** **to** **avoid** **DoS**.

## Tools

- **Burp Suite**
- **sqlmap** `--time-sec`
