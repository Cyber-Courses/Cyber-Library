---
title: "Blind LDAP injection"
description: "Inferring directory attribute values character by character through LDAP wildcard matching and true/false response differences when no data is reflected."
keywords:
  - blind LDAP injection
  - wildcard inference
  - attribute extraction
  - boolean oracle
---

# Blind

When an injectable filter does not return the matched data but the response differs between a match and no match, LDAP attributes can still be read one character at a time using the wildcard as a prefix test.

The oracle asks "does this attribute start with these characters?" by anchoring a wildcard. Against `(&(objectClass=user)(cn=$input))`, inject a target attribute with a growing known prefix:

```
input: admin)(description=S*
filter: (&(objectClass=user)(cn=admin)(description=S*))
```

A match (the "found" response) means the `description` attribute of `admin` begins with `S`; no match means it does not. Extend the prefix one character at a time (`Se*`, `Sec*`, ...) to recover the whole value. The alphabet is walked per position, so this is the LDAP analogue of boolean SQL extraction.

Presence tests enumerate which attributes exist, narrowing what to extract:

```
input: admin)(mail=*
filter: (&(objectClass=user)(cn=admin)(mail=*))
```

A match means `admin` has a `mail` attribute. Blind LDAP works only on attributes the bound account may read and that are not write-only: hashed `userPassword` is typically not returned or matchable this way, but descriptive, contact, and group attributes usually are. Like blind SQL injection, it is slow and almost always automated, but the single wildcard comparison is the reliable primitive underneath.

## Tools

- **ldapsearch**: verify prefix wildcard matches that drive the boolean oracle.
- **Burp Intruder**: automate per-character prefix extraction across the alphabet.
- **python-ldap**: script the character-by-character inference loop.

## References

- RFC 4515: LDAP search filters (substring and presence matching)
- OWASP Testing Guide: Testing for LDAP Injection
