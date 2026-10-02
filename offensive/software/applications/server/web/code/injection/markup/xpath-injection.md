---
title: "XPath injection: predicate concatenation, authentication bypass, and blind node extraction"
description: Exploiting application code that builds XPath queries by concatenating user input, closing predicates with injected quotes, adding or clauses to bypass authentication, and extracting document nodes character by character with boolean and blind techniques.
keywords:
  - XPath injection
  - XPath
  - authentication bypass
  - blind XPath
  - boolean extraction
  - XML database
---

# XPath injection

**XPath injection** occurs when an application builds an XPath query by concatenating untrusted input into the expression string and evaluates it against an XML document or XML database. Because XPath has no equivalent of prepared statements in most naive usage, an injected quote and logical operator rewrite the query's **predicate**, the filter inside `[...]`, letting an attacker bypass authentication or walk the document node by node. The technique mirrors SQL injection almost exactly, adapted to XPath's grammar.

## Overview

XML is a common backing store for user directories, configuration, and small datasets. A typical vulnerable login looks up a user by concatenating form fields into a query:

```python
# username and password come from the request
query = "//users/user[username/text()='" + username + \
        "' and password/text()='" + password + "']"
result = tree.xpath(query)
```

The developer expects two string literals. The attacker supplies a value containing a single quote, which **closes the literal early** and lets the rest of the input be parsed as XPath syntax rather than data. From that moment the attacker controls the predicate logic.

## Authentication bypass

The canonical bypass makes the predicate always true, so the first user node is returned regardless of credentials:

```
# username field:
' or '1'='1
```

The query becomes:

```
//users/user[username/text()='' or '1'='1' and password/text()='...']
```

Because `'1'='1'` is always true and `or` has lower precedence than `and`, the predicate matches every user; the application logs in as whichever node comes first (often an administrator). Variants adapt to the surrounding quoting and to how many fields are concatenated:

```
' or true() or '
']  (close the predicate entirely, where the step structure allows)
admin' or '1'='1
```

Where only the username is injectable and the password check is a separate step, closing the predicate so the password clause is neutralized (for example by commenting it out of the logic with an always-true `or`) achieves the same result.

## Quoting context

The first exploitation step is identifying how the literal is wrapped, because the breakout character differs:

- **Single-quoted literal**, `'...'`, supply a `'` to escape, e.g. `' or '1'='1`.
- **Double-quoted literal**, `"..."`, supply a `"` instead, e.g. `" or "1"="1`.
- **Numeric or unquoted context**, rarer in XPath, but a value spliced outside quotes needs no escape at all; operators work directly.

XPath 1.0 has no string-escaping mechanism inside a literal, so a value containing both quote types often cannot be represented as a single literal, an asymmetry worth probing when one quote style is filtered.

## Blind and boolean extraction

When the query result is not reflected, only a success/failure signal, such as "login succeeded" versus "failed", data is recovered by asking true/false questions, exactly as in blind SQL injection.

### Confirming a boolean oracle

```
' and '1'='1      → behaves as the normal (true) case
' and '1'='2      → behaves as the false case
```

A reliable difference between the two responses establishes the oracle.

### Walking the document

XPath exposes functions to inspect structure and content, which an attacker queries one bit at a time:

```
# how many child nodes under the current context:
' and count(/*)=1 and '1'='1

# length of the first user's name:
' and string-length((//user[1]/username))=5 and '1'='1

# character-by-character, comparing each position:
' and substring((//user[1]/password),1,1)='a' and '1'='1
```

Iterating `substring(...)` over positions and candidate characters extracts arbitrary node text, usernames, password hashes, any element in the document. `name(...)`, `count(...)`, and `local-name(...)` enumerate unknown structure first, so the extraction targets real paths.

### Blind without a visible boolean

If no explicit success/failure text differs, a secondary signal substitutes: response length, an error that appears only on malformed evaluation, or timing where the backend is slow enough. The inference loop is identical; only the oracle changes.

## XPath version differences

- **XPath 1.0** is the most common target: string functions `substring`, `string-length`, `contains`, `starts-with`, `concat`, and `count` are the extraction toolkit. There is no `lower-case` or regex.
- **XPath 2.0 / 3.1** add `matches()` (regex), `lower-case()`, `string-join()`, and richer sequence handling, which streamline extraction and case-insensitive matching when the engine supports them. Fingerprinting the version, by testing whether a 2.0-only function evaluates, decides which payloads are available.

## Exploitation workflow

1. **Probe for a break.** Submit a lone quote (`'`) and watch for an XPath error or changed behavior that signals the literal was broken.
2. **Fix the context.** Determine single vs double quoting and how many clauses are concatenated.
3. **Try the bypass.** For auth forms, an always-true `or` is the fastest confirmation and often the whole exploit.
4. **Build the oracle.** For data extraction, establish a stable true/false signal.
5. **Extract.** Enumerate structure with `count`/`name`, then pull text with `substring` and `string-length`.

## Tools

- **[Burp Suite](https://portswigger.net/burp)** (Repeater for manual probing, Intruder to automate the `substring` character sweep).
- **[xcat](https://github.com/orf/xcat)** automates blind and OOB XPath extraction, including structure enumeration and character-by-character retrieval.
- A local XML document and XPath engine matching the target to validate quoting and function availability before sending payloads at the application.

## References

- [CWE-643: Improper Neutralization of Data within XPath Expressions ('XPath Injection')](https://cwe.mitre.org/data/definitions/643.html)
- [OWASP: XPath Injection](https://owasp.org/www-community/attacks/XPATH_Injection)
- [OWASP WSTG: Testing for XPath Injection](https://owasp.org/www-project-web-security-testing-guide/)
- [PayloadsAllTheThings: XPATH Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/XPATH%20Injection)
