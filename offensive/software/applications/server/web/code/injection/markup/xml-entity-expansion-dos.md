---
title: "XML entity expansion DoS: billion laughs and quadratic blowup"
description: Denial of service against XML parsers that still process DTDs, recursive entity references (billion laughs) and large repeated expansions (quadratic blowup) that inflate a tiny upload into gigabytes of memory and CPU.
keywords:
  - XML bomb
  - billion laughs
  - quadratic blowup
  - entity expansion
  - denial of service
  - XXE
---

# XML entity expansion (DoS)

**Entity-expansion** attacks abuse a legitimate XML feature, internal general entities, to amplify a few hundred bytes of input into a structure large enough to exhaust the parser's memory or CPU. Unlike [XXE](xml-external-entity-xxe.md), there is no data theft and no outbound request; the payload never leaves the document. The impact is purely **availability**: a single small request can stall a worker, spike memory until the process is killed, or pin a core until a request timeout fires.

## Overview

XML lets a document define an entity and reference it many times; the parser substitutes the entity's value at each reference. The grammar permits entities to reference *other* entities. When those two facts combine without a cap on total expanded size, a small set of nested definitions expands geometrically. The attack is viable whenever the target parser **still processes DTDs and expands internal entities**, the same configuration surface as XXE, but exploiting expansion rather than external resolution.

## Billion laughs

The classic payload nests entities so each layer multiplies the one below it. Ten levels of ten references produces 10^9 copies of the base string:

```xml
<?xml version="1.0"?>
<!DOCTYPE lolz [
  <!ENTITY lol "lol">
  <!ENTITY lol1 "&lol;&lol;&lol;&lol;&lol;&lol;&lol;&lol;&lol;&lol;">
  <!ENTITY lol2 "&lol1;&lol1;&lol1;&lol1;&lol1;&lol1;&lol1;&lol1;&lol1;&lol1;">
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

The source is under a kilobyte; the expanded `&lol9;` is roughly three gigabytes of `lol`. A parser that materializes the expanded tree in memory is forced to allocate until it is killed or the host swaps to a crawl. Depth and width are tunable, fewer levels or narrower fan-out produce a payload calibrated to stall without instantly triggering a crash that a supervisor would restart.

## Quadratic blowup

Deeply nested definitions are a well-known signature, so some parsers cap nesting depth or total entity count. **Quadratic blowup** sidesteps depth limits using a *single* large entity referenced many times at one level:

```xml
<?xml version="1.0"?>
<!DOCTYPE bomb [
  <!ENTITY a "AAAAAAAAAA... (tens of thousands of characters) ...AAAA">
]>
<bomb>&a;&a;&a;&a;&a; ... (tens of thousands of references) ... &a;</bomb>
```

With an entity of length *n* referenced *n* times, the parser does O(n²) work and allocates O(n²) bytes from an input only O(n) in size. There is no recursion to detect; the document is flat, defeating depth-based guards while still achieving large amplification. Tuning *n* trades request size against expansion.

## External-entity amplification

Where a parser resolves external parameter entities, expansion can be combined with a remote DTD so the *definition* of the exploding entity is fetched rather than inlined, shrinking the on-the-wire payload and moving the amplification server-side. This overlaps with the DTD mechanics in [XXE](xml-external-entity-xxe.md); only the objective, resource exhaustion, differs.

## Exploitation notes

- **Calibrate, do not sledgehammer.** In an authorized test the goal is to *prove* the parser expands unbounded, not to take production down. Start with a modest multiplier (a few million expansions) and measure response latency and memory before scaling, the proof of concept is a measurable CPU/RAM spike, not a sustained outage.
- **Pick the variant to the defense.** Billion laughs demonstrates recursive expansion; quadratic blowup demonstrates the flat case that bypasses nesting caps. Testing both shows which guard (if any) is present.
- **Delivery mirrors XXE.** Any endpoint that parses XML, SOAP, SVG upload, Office documents, XML APIs, carries the payload, since the vulnerable toggle (DTD + entity expansion) is the same.
- **Confirmation signals.** Rising response time with payload size, a worker process OOM-killed in logs, or a request that hits the server's timeout while a baseline request is instant all confirm unbounded expansion.

## Tools

- **[Burp Suite](https://portswigger.net/burp)** Repeater to submit staged payloads and compare response timing and failure modes.
- A local build of the target's XML library to confirm whether entity expansion and DTD processing are enabled before sending anything at the application.
- Simple scripts to generate billion-laughs and quadratic payloads at a chosen multiplier, so amplification is dialed in rather than guessed.

## References

- [CWE-776: Improper Restriction of Recursive Entity References ('XML Bomb')](https://cwe.mitre.org/data/definitions/776.html)
- [CWE-400: Uncontrolled Resource Consumption](https://cwe.mitre.org/data/definitions/400.html)
- [OWASP: XML Entity Expansion / Denial of Service](https://owasp.org/www-community/vulnerabilities/XML_Entity_Expansion)
- [PayloadsAllTheThings: XXE Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/XXE%20Injection)
