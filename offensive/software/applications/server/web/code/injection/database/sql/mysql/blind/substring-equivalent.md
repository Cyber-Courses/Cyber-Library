---
title: "Substring, MID, and ORD-based extraction in blind MySQL SQL injection"
description: SUBSTRING, LEFT, MID, and LOCATE-based comparisons for blind extraction without echoing full rows.
keywords:
  - SUBSTRING SQL injection
  - blind SQLi
  - MySQL
---

# Substring extraction

## Context

`SUBSTRING(str,pos,len)`, `LEFT`, `RIGHT`, and `MID` isolate **one** **character** or **byte** for **`=`**, **`>`**, or **range** **tests** inside `IF` or **`AND`**. This is the **workhorse** for **boolean** **blind** **extraction** when **no** **error** or **union** **channel** exists.

## Theory

Compare **efficiency**: **binary** **search** on **ASCII** **value** vs **alphabet** **iteration**. For **long** **secrets**, **bitwise** **`&`** **tests** reduce **round** **trips**. **`ORD()`** and **`ASCII()`** normalize **characters** to **integers**.

## Practice

### Midpoint search on ORD

- For position `i`, binary-search `ORD(SUBSTRING(secret,i,1))` between 0 and 127 (or 255 for binary columns).

### Known-length secrets

- If length leaks (`LENGTH(secret)=32` for MD5 hex), loop positions **1..32**.

## Tools

- **sqlmap**
- **Burp Suite**
