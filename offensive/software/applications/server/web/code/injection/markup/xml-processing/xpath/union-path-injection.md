---
title: "XPath union path injection"
description: "The | union operator appends arbitrary location paths to a string-built XPath, selecting nodes outside the intended subtree and dumping the whole document."
keywords:
  - XPath union injection
  - XPath pipe operator
  - location path injection
  - node-set union
  - XML document dump
---

# Union path injection

XPath's `|` operator returns the union of two node-sets. Where SQL injection reaches for `UNION SELECT`, XPath injection reaches for `|`, and it is simpler: there is no column count to match and no schema to satisfy, so an injected pipe can append any location path and fold its results into the query output.

## The operator

Given a query that returns a constrained node-set, closing the predicate and appending `| <path>` adds a second, attacker-chosen path. A query such as `//user[name='x']/pass` can be turned into a union that also returns unrelated nodes:

```
x']|//*|//user['
```

The expression becomes `//user[name='x']|//*|//user['']/pass`. The middle term `//*` selects every element in the document, so the result set now contains the entire tree rather than one password node. The leading and trailing fragments exist only to keep the surrounding syntax well formed.

## Dumping the whole document

`//*` is the broadest selector and returns every element node. When the application reflects query results into the page, this prints the document:

```
']|//*|//foo[bar='
```

For a more readable dump, target text and attribute nodes explicitly:

```
']|//*/text()|//foo[bar='
']|//@*|//foo[bar='
```

`//@*` enumerates every attribute across the document, which often holds roles, identifiers, and flags that are not present as element text.

## Reaching specific subtrees

Union does not have to be indiscriminate. Append a precise path to pull one branch of interest while keeping the original query valid:

```
']|//user/pass|//foo[bar='
']|//config/credentials|//foo[bar='
```

Each injected path is independent of the original query's constraints, so a filter that limited the first node-set to a single user is irrelevant to the unioned path.

## Parent and ancestor axes

When the injection point sits deep in the tree, the parent (`..`) and ancestor axes climb back toward the root to reach sibling data the original query walked past. This recovers nodes above or beside the intended selection:

```
']|../../*|//foo[bar='
']|ancestor-or-self::*|//foo[bar='
']|//user[1]/following-sibling::user|//foo[bar='
```

`following-sibling` and `preceding-sibling` step across records at the same level, which is an efficient way to enumerate every `user` once one is reachable. The ancestor axes are the vertical complement: from any matched node they expose its containers up to the document root.

## Keeping the expression valid

The recurring pattern is `<close> | <injected path> | <reopen>`, where the closing fragment terminates the original literal and predicate and the reopening fragment supplies a dummy predicate the parser can finish. If the application rejects the request outright rather than returning extra nodes, the balance is wrong; count the open quotes and brackets in the original query and mirror them. A malformed union typically yields a parser error, which error-based extraction can then exploit in its own right.

## Tools

- **Burp Repeater**: appending `|` union paths and reading the extra nodes.
- **xcat**: automating node-set enumeration across the document.
- Manual testing with `|//*` and axis-based union payloads.

## References

- [OWASP: XPATH Injection](https://owasp.org/www-community/attacks/XPATH_Injection)
- [PayloadsAllTheThings: XPATH Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/XPATH%20Injection)
