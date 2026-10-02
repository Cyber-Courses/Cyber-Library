---
title: "Authentication bypass through LDAP injection"
description: "Bypassing an LDAP-backed login that builds its search filter from the username and password, using the wildcard and injected boolean logic."
keywords:
  - LDAP authentication bypass
  - userPassword wildcard
  - login bypass
  - filter injection
---

# Authentication bypass

Applications that authenticate against a directory often build a search filter from the submitted credentials and treat a matching entry as a successful login. When the fields are not escaped, the filter logic can be subverted.

Take a login that searches with `(&(uid=$user)(userPassword=$pass))` and logs in if one entry matches. The simplest bypass uses the wildcard in the password field, which matches any stored value when the password attribute is present and compared in the filter:

```
user: admin
pass: *
filter: (&(uid=admin)(userPassword=*))
```

This returns the admin entry regardless of the real password. Where the match is on the username side, injected logic forces an always-true filter:

```
user: *)(uid=*))(|(uid=*
filter: (&(uid=*)(uid=*))(|(uid=*)(userPassword=...))
```

This leaves two adjacent top-level filters (`(&(uid=*)(uid=*))` and `(|...)`) rather than one valid RFC 4515 filter, so it works only against a permissive client library that uses the first complete filter and ignores the rest; a client that validates the whole string rejects it with a bad-filter error. A single-clause filter `(uid=$user)` is easier and always valid: `user=*` returns the first entry.

The wildcard-password form is the most reliable, and it works only when the application compares the password inside the filter rather than performing a separate bind with the supplied password (a bind-based login hashes and checks server-side, so the wildcard does not help there). Establish which pattern the app uses by testing the wildcard first.

## Tools

- **ldapsearch**: replay candidate filters against the directory to confirm a wildcard or boolean bypass.
- **python-ldap**: script bind and search attempts to find a working bypass payload.
- **Burp Repeater**: craft and resend tampered username/password login requests.

## References

- RFC 4515: LDAP search filters
- OWASP Testing Guide: Testing for LDAP Injection
