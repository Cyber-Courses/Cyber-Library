---
title: "Parameter omission in workflows: optional fields that gate authorization, state, or pricing logic"
description: Triggering unintended server branches by omitting, nulling, or stripping parameters optional in the schema but used for authorization or workflow gates.
keywords:
  - parameter omission
  - optional parameter
  - workflow
---

# Parameter omission

## Context

**Parameter omission** exploits **multi**-**action** **handlers** and **default** **branch** **logic**: if **`approve`** is **absent**, the **code** **path** **may** **default** to **approve**; if **`amount`** is **missing**, a **fee** **waiver** **branch** **may** **run**. Distinct from **step skip** (different **URL**) and from **BOLA** on **ids** (this is **presence** of **fields**, not **wrong** **id**).

## Theory

**JSON** **merge**, **PATCH**, **and** **form** **encoding** **differ** on **null** **vs** **missing** **vs** **empty** **string**. **Server** **code** that **does** `if (body.approve === true)` **without** **an** **`else`** **reject** **may** **treat** **omitted** **`approve`** as **not** **false**.

## Practice

### One-parameter-at-a-time drop

- In a proxy, **remove** **each** **JSON** **key** **from** a **state**-**changing** **request** **and** **diff** **responses** **and** **side** **effects** on **test** **data**.

### Empty vs missing vs null matrix

- Send **`{}`**, **`{"approve":null}`**, **`{"approve":""}`** for **the** **same** **endpoint** **and** **record** **which** **combination** **crosses** **a** **gate**.

## Tools

- **Burp Suite**
- **curl**
