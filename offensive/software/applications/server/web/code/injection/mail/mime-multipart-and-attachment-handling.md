---
title: "MIME multipart and attachment handling: boundary injection and type confusion"
description: Unsafe assembly of MIME messages where user input influences boundaries, filenames, or nested multiparts.
keywords:
  - MIME
  - email injection
  - multipart
---

# MIME multipart issues

## Context

Libraries build `multipart/mixed` with **attacker-controlled** filenames or **boundary** strings. Weak patterns allow **closing** one part early or **smuggling** extra parts that parsers interpret differently than the human reader.

## Theory

Use library APIs that generate **random** boundaries and **encode** filenames; never concatenate raw MIME with unescaped user text.

## See also

- [Mail injection (parent)](index.md)
- [SMTP header injection](smtp-header-injection-crlf.md)
