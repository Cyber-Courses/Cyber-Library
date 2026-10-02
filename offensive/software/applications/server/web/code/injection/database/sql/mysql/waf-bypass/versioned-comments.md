---
title: "MySQL versioned and inline comments for filter bypass"
description: "Using MySQL /*! versioned */ and /**/ inline comments to split blocked keywords and conditionally execute SQL past signature filters."
keywords:
  - versioned comment
  - inline comment bypass
  - keyword splitting
  - MySQL filter evasion
---

# Versioned comments

MySQL treats `/*! ... */` as a versioned comment: the contents are ignored by other databases but executed by MySQL, and with an embedded version number like `/*!50000 ... */` they run only on MySQL at or above that version (5.0.0 here). This is a filter-evasion staple because the keyword hides inside what looks like a comment.

Splitting a blocked keyword across versioned comments defeats a literal `UNION SELECT` signature while still executing:

```sql
' /*!50000UNION*/ /*!50000SELECT*/ 1,2,3-- 
```

Plain inline comments `/**/` replace whitespace, so a filter looking for the space-separated `UNION SELECT` token misses the run-together form:

```sql
'/**/UNION/**/SELECT/**/1,2,3-- 
```

The two combine freely, and comments can also be inserted mid-keyword only inside the versioned form (a bare `UN/**/ION` is not valid, but `/*!UNION*/` is). Because the version gate silently drops the payload on servers below the number, set it low (or omit it) unless you are deliberately fingerprinting the version by toggling execution on and off.

## References

- MySQL Reference Manual: comment syntax, versioned comments
- OWASP Testing Guide: Testing for SQL Injection
