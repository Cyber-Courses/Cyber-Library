---
title: "TLS client certificate misbinding: wrong identity bound to HTTP sessions or upstream context"
description: mTLS on the wrong network hop or a weak mapping from certificate identity to account rows in application code.
keywords:
  - mTLS
  - client certificate
  - certificate mapping
  - identity binding
---

# Client certificate misbinding

## Context

Mutual TLS can authenticate the *connection*, but the app must map the presented identity to **one** user or service account. Misbinding is: the proxy verified the cert and passed a header the app misreads, or the same cert maps to multiple tenants, or the app treats “any client cert” as “user is trusted.”

## Theory

The offensive signal is a valid cert for a test identity A used on a code path that loads user B’s session because the mapping table or header name is wrong. This is not the same as forged headers without mTLS; the cert is real but the *binding* to application identity is wrong.

## Practice

### Hold two test certs in a lab mesh

- With two client certs issued for different subjects, call the same authenticated route and compare which `sub` or account id the app sets. Swap order of `ssl_client_s_dn` style variables if the stack exposes them to the app in inconsistent ways.

## Tools

- **curl** with `--cert` and `--key`
- **OpenSSL** `s_client`
- **Burp Suite** (with client cert loaded for in-scope test)
