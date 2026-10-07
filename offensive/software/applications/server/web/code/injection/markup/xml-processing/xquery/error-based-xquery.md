---
title: "Error-based XQuery: inferring data from surfaced errors"
order: 3
description: "XQuery static and dynamic errors returned to the client leak content, types, and structure through controlled error conditions and embedded data in error strings."
keywords:
  - error-based XQuery
  - XQuery error messages
  - dynamic error inference
  - type error leak
  - blind XQuery extraction
---

# Error-based

When an application returns processor errors to the client, XQuery injection does not need reflected results to extract data. The error text, the error code, and the mere presence or absence of an error each carry information, and an attacker shapes the injected expression so that the condition being tested decides which of those outcomes occurs. This turns a single request into a reliable oracle even on endpoints that render nothing but a generic failure page, provided the status or body differs between success and error.

## Data embedded in the error string

The most direct technique forces the engine to place target data inside a message it then prints. Casting a stolen string to a type that cannot hold it makes the processor quote the offending value in a dynamic error:

```xquery
' or (//user[1]/password) cast as xs:integer or '
```

Because a password is not a valid integer, the engine raises `FORG0001` and many deployments include the rejected value in the message, leaking the secret verbatim. `fn:error()` with a constructed description is even more explicit where the application forwards custom error text:

```xquery
' or error(xs:QName('err:LEAK'), string-join((//user/password), '|')) or '
```

## Conditional error as a boolean oracle

When messages are scrubbed but errors still change the response, a predicate is made to trigger a guaranteed runtime fault only on a true condition. Division by zero and out-of-range subsequence access are convenient faults:

```xquery
' or (1 div (if (substring((//user[1]/password),1,1)='a') then 0 else 1)) or '
```

A true guess divides by zero and raises `FOAR0001`, yielding the error page; a false guess evaluates cleanly and returns the normal page. Iterating `substring()` across positions reconstructs the value one character per request, the same search used in blind boolean injection but keyed on the error signal instead of the result set.

## Type and structure probing

Static type errors reveal the shape of the data without extracting it. Applying a node-only function to a bound item, or comparing incompatible types, tells the attacker whether a path yields an element, an atomic value, or an empty sequence:

```xquery
' or name(//user[1]/*[1]) = 'x' or fn:local-name(0) or '
```

The specific code returned (`XPTY0004` for a type mismatch, `XPST0017` for an unknown function or wrong arity) also fingerprints the engine and its XQuery version, since BaseX, eXist-db, Saxon, and MarkLogic emit distinct wording and prefixes around the same W3C codes. That fingerprint guides which vendor extension functions to attempt next.

## Vendor error strings

Engine-specific messages frequently disclose filesystem paths, module URIs, and internal variable names. A deliberately malformed `doc()` or `file:read-text()` call provokes a not-found error whose text echoes the resolved absolute path, mapping the deployment layout for a follow-up file-read payload.

## Tools

- **Burp Repeater**: forcing cast and division-by-zero faults and reading error text.
- Manual testing with `cast as xs:integer` and `fn:error()` payloads.

## References

- [W3C XQuery 3.1: Error Handling](https://www.w3.org/TR/xquery-31/#id-error-handling)
- [PayloadsAllTheThings: XPath Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/XPATH%20Injection)
