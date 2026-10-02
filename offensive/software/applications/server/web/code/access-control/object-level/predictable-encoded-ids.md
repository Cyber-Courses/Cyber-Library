---
title: "Predictable encoded IDs: reversible encodings of database keys in URLs and tokens"
description: Obfuscated identifiers (Base64, custom encodings) used in place of authorization, which only slow manual testing rather than replace access decisions.
keywords:
  - encoded id
  - obfuscation
  - IDOR
---

# Predictable encoded IDs

## Context

Encoding `user_id=7` as Base64 or a custom scheme hides the value from casual eyes but is not an access control. Decoding in a script and replaying the same BOLA test as a raw id is the same finding with one extra step.

## Theory

Double encoding, URL encoding, and short fixed-width schemes reduce online guessing space; they do not add a server-side check. Client-side only generation of “opaque” tokens is still guessable if the algorithm is in the mobile binary.

## Practice

### Decode and replay in a lab

- Copy an id from traffic, Base64-decode or reverse the visible scheme, modify the integer or string inside, re-encode, and substitute in the same API call. If access tracks the modified target, the server never enforced ownership—only obscurity.

## Tools

- **CyberChef** (in lab)
- **Burp Suite**
- **Python** one-liner in a test shell
