---
title: "XQuery injection"
description: "String-built XQuery against XML databases lets an attacker rewrite path expressions and predicates, reach the filesystem and network through built-in functions, and read data blindly."
keywords:
  - XQuery injection
  - XML database injection
  - BaseX
  - eXist-db
  - MarkLogic
---

# XQuery

XQuery is the query language for XML databases such as **BaseX**, **eXist-db**, and **MarkLogic**, and it is injectable for the same reason SQL is: when application code concatenates user input into a query instead of binding external variables, the input is parsed as query syntax. An attacker who reaches a string-built `for`, `where`, or predicate can break out of a string literal, alter the XPath navigation, and append further expressions.

Because an XQuery processor walks a document or collection with no column boundaries, a single injected expression such as `//*` can enumerate every node in scope. The language also ships a rich function library: `doc()`, `fn:collection()`, and `fn:unparsed-text()` reach files and URLs, and vendor extensions (BaseX `proc:system`, `file:read-text`) reach the filesystem and shell. Where no data is reflected, dynamic and static errors surfaced to the client support blind inference.

This subtree covers FLWOR and predicate breakout, library and extension function abuse, and error-based extraction.
