---
title: "Data: offensive manipulation of encodings, ciphers, and transforms"
description: "The data category covers attacks on how information is represented and transformed, from character encodings and binary formats to ciphers and reversible transforms, independent of the application that carries it."
keywords:
  - data manipulation
  - encoding attacks
  - cipher attacks
  - binary data
  - data transformation
---

# Data

The data category covers offensive work against the way information is represented and transformed, below the level of any single application. The same bytes flow through encoders, ciphers, and serializers on their way between systems, and each of those steps is an attack surface in its own right: a decoder that trusts its input, a cipher used without integrity, or a transform that is reversible when it was assumed not to be.

## Why it is split this way

The subcategories group by the kind of representation or transformation under attack, because the techniques and tools differ sharply between them:

- **Encoding**: reversible schemes that carry data without secrecy (Base64, URL, percent, hex). The offensive angle is smuggling, filter evasion, and parser confusion.
- **Character encoding**: charset and Unicode handling, where normalization, overlong forms, and homoglyphs defeat comparisons and validators.
- **Ciphers**: cryptographic transforms, attacked through misuse rather than math, for example missing integrity, predictable IVs, and oracle behavior.
- **Binary data**: raw and structured binary formats, hex-level manipulation, and format-aware tampering.
- **Data transformation**: compression, serialization, and format conversion, where the transform itself (not the payload) carries the flaw.

Splitting on representation keeps each page focused on one class of parser or algorithm, so a technique that abuses Unicode normalization does not get tangled with one that abuses a block cipher mode. These are foundational primitives that recur across the [software](../software/index.md), [network](../network/index.md), and application layers, which is why they live in their own category rather than under any one target.

## References

- [OWASP: Testing for Weak Cryptography](https://owasp.org/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/09-Testing_for_Weak_Cryptography/)
- [OWASP: Injection and encoding](https://owasp.org/www-community/attacks/)
