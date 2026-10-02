---
title: "Public identifier guessing: sequential IDs, short slugs, and enumerable public references"
description: Unlisted but guessable slugs, shortlink tokens, and public share URLs that return data without an entitlement check in code.
keywords:
  - unlisted URL
  - slug
  - shortlink
  - BOLA
---

# Identifier guessing

## Context

“Anyone with the link” often means a long unguessable token. When the token is a short English slug or a low-entropy string, discovery is enumeration, not magic. The server must still implement a server-side rule for who may use a link, not only obscurity.

## Theory

Search engines, referrers, and public indexes may already list some “unlisted” pages. The offensive question is what happens when a second subject requests the same slug with a different session or no session.

## Practice

### Walk slug space in test programs

- In a lab, generate objects with adjacent slugs or incremental numeric public codes. Request neighbors to see if unowned slugs return `404` consistently or leak metadata.

## Tools

- **ffuf**
- **curl**
- **Burp Suite**
