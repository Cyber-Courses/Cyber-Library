---
title: "Internationalized domain and normalization bypasses"
description: "Unicode host labels, punycode, and normalization let a hostname read as an allowed domain to one check and resolve as an attacker domain to another, slipping past string-based allowlists."
keywords:
  - internationalized domain
  - punycode
  - IDN homograph
  - unicode normalization
  - NFKC
  - hostname allowlist bypass
---

# Internationalized domain

Internationalized domain names let hostnames contain Unicode. Between the string a validator compares and the name a resolver looks up sit two transformations: normalization (such as NFKC) and the IDNA `toASCII` conversion to punycode. When the validator and the client apply these at different points, a hostname can read as an allowed domain to the check and resolve as an attacker-controlled domain to the fetch.

## Normalization that rewrites the host

Unicode normalization maps many characters to ASCII equivalents. A host that contains a compatibility character passes a check that runs before normalization, then becomes a different name once the client normalizes it. For example, the fullwidth and compatibility forms below normalize under NFKC toward ASCII letters, so a validator comparing raw bytes sees something other than what the resolver ultimately uses:

```
ⓔⓧⓐⓜⓟⓛⓔ.com        # enclosed alphanumerics -> example.com under NFKC
ｅｘａｍｐｌｅ.com        # fullwidth Latin -> example.com under NFKC
```

If the allowlist check runs on the pre-normalization string and the HTTP client normalizes before resolving, the two disagree.

## Homographs and the allowed-label trick

A confusable character makes a hostname display like an allowed domain while being a distinct name that the attacker registers and controls. A Cyrillic `а` (U+0430) in place of Latin `a` produces a visually identical but different domain that resolves to the attacker:

```
exаmple.com          # the third letter is Cyrillic U+0430, not Latin a
```

Any label with a non-ASCII character has an equivalent `xn--` punycode form produced by `toASCII` (for instance the ASCII-safe encoding of `bücher` is `xn--bcher-kva`). An allowlist comparing against the Unicode form can be satisfied by a homograph; one comparing the punycode form must match the exact `xn--` label. The gap is a mismatch in which form each side uses to compare.

## Punycode and the two-form mismatch

The same host exists as a Unicode string and as its `xn--` punycode encoding, and different components prefer different forms. A validator that lowercases and compares the Unicode form, paired with a resolver that operates on punycode (or vice versa), can be driven to approve one representation and resolve another. Supplying the host in whichever form the validator does not canonicalize is the core move: send the Unicode label where the check expects punycode, or the `xn--` label where it expects Unicode, so the comparison and the resolution disagree.

## Combining with the resolved target

Internationalized-domain tricks get the request past a *name* check; the host still has to resolve to something useful. Point the attacker-controlled name at an internal address (or pair it with [DNS rebinding](dns-rebinding.md)) so that clearing the allowlist lands the connection on `127.0.0.1`, an RFC1918 host, or the metadata service. Where the allowlist canonicalizes both sides identically to punycode before comparing, this avenue closes and a raw [IP address](../ip-address.md) encoding is the fallback.

## References

- [OWASP: Server Side Request Forgery](https://owasp.org/www-community/attacks/Server_Side_Request_Forgery)
- [Unicode Technical Standard 46: IDNA](https://www.unicode.org/reports/tr46/)
