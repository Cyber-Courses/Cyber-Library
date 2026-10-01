---
title: "Internationalized domain and normalization bypasses"
description: "Unicode host labels and normalization sit between the string an SSRF filter inspects and the name the client resolves, so a lookalike that normalizes to a blocked host, or a validator and resolver that normalize differently, defeat a string-based control."
keywords:
  - internationalized domain
  - punycode
  - unicode normalization
  - NFKC
  - SSRF blocklist bypass
  - hostname validation differential
---

# Internationalized domain

Internationalized domain names let hostnames contain Unicode. Between the string a validator compares and the name a resolver looks up sit two transformations: Unicode normalization (such as NFKC) and the IDNA `toASCII` conversion to punycode. SSRF-relevant bypasses come from a mismatch in *when* and *how* those transformations are applied, not from a lookalike by itself: an exact-match control compares strings, and a confusable character simply produces a different string that the check rejects. The useful cases are where normalization maps an attacker string onto a host the policy cares about, or where the validator and the client normalize differently.

## Blocklist bypass by normalization

The strongest case is a **blocklist** keyed on an ASCII literal such as `localhost` or a blocked internal name. Unicode normalization maps many compatibility characters to ASCII, so a hostname that is not byte-equal to the blocked string can still normalize to it. A validator that matches the raw bytes sees something other than `localhost` and allows it; the client then normalizes and resolves the blocked name, reaching the internal host:

```
ｌｏｃａｌｈｏｓｔ        # fullwidth Latin, NFKC-normalizes to localhost
ⓛⓞⓒⓐⓛⓗⓞⓢⓣ        # enclosed alphanumerics, NFKC-normalizes to localhost
```

The same applies to any internal hostname a blocklist tries to exclude: supply a compatibility-character spelling that passes the string check and normalizes back to the forbidden name.

## Validator and resolver differential

Against an **allowlist**, the requirement is a difference in how the two stages canonicalize. When the validator and the HTTP client apply different normalization, different IDNA processing, or compare in different forms (one Unicode, one `xn--`), a single hostname can satisfy the check while resolving elsewhere. The attacker supplies the host in whichever form the validator does not canonicalize the way the client does:

```
аllowed.example      # a Cyrillic lookalike that a flawed validator folds toward the allowed label
xn--...              # or the A-label form, where the validator compares Unicode but the client resolves punycode
```

This only works where such a canonicalization gap exists; without it, the lookalike is a distinct name the allowlist rejects. The attacker-registered name must also resolve to the internal target, so pair it with a resolution the attacker controls (or with [DNS rebinding](dns-rebinding.md)).

## Homographs are visual, not a bypass on their own

A confusable such as a Cyrillic `а` (U+0430) for Latin `a` makes a hostname look like an allowed domain, which matters for phishing but not for a string comparison: the homograph is a different name and an exact check rejects it. It aids SSRF only in combination with one of the gaps above (a blocklist that normalizes, or a validator/resolver differential). Treat the lookalike as the delivery and the comparison flaw as the vulnerability.

## References

- [OWASP: Server Side Request Forgery](https://owasp.org/www-community/attacks/Server_Side_Request_Forgery)
- [Unicode Technical Standard 46: IDNA](https://www.unicode.org/reports/tr46/)
