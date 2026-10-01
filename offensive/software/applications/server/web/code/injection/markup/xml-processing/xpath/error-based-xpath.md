---
title: "Error-based XPath extraction"
description: "Forcing XPath type and syntax errors makes the processor leak node text and structure in its error message, turning a verbose parser into a direct data channel."
keywords:
  - error-based XPath
  - XPath error message
  - string coercion
  - XPath data extraction
  - XML error leak
---

# Error-based

When an XPath processor returns its error messages to the client, those messages become an extraction channel. By deliberately forcing a type conversion or syntax fault whose message embeds node text, an attacker reads values directly out of the error instead of out of the page, which turns a verbose processor into a fast, non-blind oracle.

## Why errors leak data

Many XPath and XQuery engines include offending content in their diagnostics. An expression that tries to use a node-set where a number is required, or that calls a function with a malformed argument built from node text, raises an error whose text often contains the coerced value. The extraction trick is to arrange the expression so the value you want is what gets interpolated into the message.

## Forcing a type error with coercion

Coercing a string into a numeric context is the classic trigger. Appending arithmetic to a `string()` of a target node forces the engine to parse that string as a number, and the failed conversion reports the string:

```
' or string-length(name(//user[1]))=1 and string(//user[1]/pass) + 1 or '
```

Engines that echo the non-numeric operand surface the password text in the error. A more direct form passes node text into a function that validates its argument:

```
' and extractvalue(1, concat(0x7e, (//user[1]/pass)))='
```

`extractvalue`, where the backend exposes it, reports its malformed second argument verbatim, prefixed by the `~` (`0x7e`) marker, which cleanly delimits the leaked value in the response.

## Leaking structure

Before pulling values, the same mechanism reveals the shape of the document. `name()` and `local-name()` return the element name of a node, and forcing that name into an error exposes tag names one node at a time:

```
' or string(name(/*[1])) = error() or '
' and count(//*) = 'x' or '
```

Converting a node count to a bad type reports the count; reading `name(/*[1])` names the root element. Walking indices maps the full tree structure, which then guides where to aim value extraction.

## Walking values out

With structure known, step through target nodes using `substring` to keep each leaked fragment small and unambiguous, pushing each fragment into the faulting function:

```
' and extractvalue(1, concat(0x7e, substring((//user[1]/pass),1,20)))='
```

Advance the `substring` offset to read past the first twenty characters, and change the path to move between nodes:

```
' and extractvalue(1, concat(0x7e, substring((//user[2]/@role),1,20)))='
```

Where no `extractvalue`-style function exists, division by a string or an invalid cast still produces a type error that names the operand:

```
' or 1 div string(//user[1]/pass) or '
```

## Generic syntax faults for fingerprinting

Even when a message does not carry data, its exact wording identifies the processor (libxml2, MSXML, Saxon, .NET), and that in turn tells you which functions and axes are available. A deliberately malformed expression is the quickest fingerprint:

```
'
"]
count(//
```

An unbalanced quote or bracket returns an engine-specific parse error. Match subsequent payloads, such as whether `extractvalue` or XPath 2.0 functions are usable, to the engine the error names.

## References

- [OWASP: XPATH Injection](https://owasp.org/www-community/attacks/XPATH_Injection)
- [OWASP WSTG: Testing for XPath Injection](https://owasp.org/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/07-Input_Validation_Testing/09-Testing_for_XPath_Injection)
- [PayloadsAllTheThings: XPATH Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/XPATH%20Injection)
