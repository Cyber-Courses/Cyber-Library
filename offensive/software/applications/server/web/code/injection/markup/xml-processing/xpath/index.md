---
title: "XPath injection"
description: "String-built XPath over an XML document lets an attacker rewrite predicates and location paths to bypass authentication, select unintended nodes, and extract the whole document."
keywords:
  - XPath injection
  - XML query injection
  - predicate injection
  - XPath authentication bypass
  - blind XPath
---

# XPath

XPath is the query language for navigating an XML document, and it is injectable for the same reason SQL is: when application code concatenates user input into an expression instead of binding it, the input is parsed as query syntax. An attacker who reaches a string-built XPath can break out of a string literal, rewrite the predicate, and redirect the location path to nodes the query was never meant to return.

A common vulnerable shape is authentication against an XML user store:

```python
expr = "//user[name='" + username + "' and pass='" + password + "']"
root.xpath(expr)
```

With `username` set to `' or '1'='1`, the predicate always holds and the query returns a user node regardless of the password. XPath has no access control and no notion of separate tables: the whole document is one addressable tree, so a single injected expression can walk from the authentication node to every other node, label, and attribute in the file.

XPath injection is in several ways more permissive than SQL injection. There is no concept of a privileged versus unprivileged connection, comments are rarely needed because expressions are short, and the `|` union operator joins arbitrary location paths with no schema to satisfy. Where results are not reflected, boolean and error-based inference reconstruct the document character by character.

This subtree covers predicate injection and authentication bypass, union path injection across the tree, and error-based extraction.
