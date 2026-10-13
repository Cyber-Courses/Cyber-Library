---
title: "Authentication bypass: reaching WebDAV verbs past weak auth"
order: 1
description: "WebDAV authentication is frequently inconsistent: some verbs are protected while others are not, HTTP verb tampering reaches a protected path through an unchecked method, and weak or default credentials guard the rest. Bypassing or guessing past this access control exposes the write verbs that lead to file planting and code execution."
keywords:
  - webdav auth
  - verb tampering
  - http method
  - default credentials
  - access control
---

# Authentication bypass

WebDAV access control is often applied unevenly, and that inconsistency is the bypass. A common flaw is method-scoped protection: a server configured to require authentication for `GET`/`POST` may leave `PUT`, `MOVE`, or `PROPFIND` unprotected, so an attacker reaches a protected resource through an unchecked verb (HTTP verb tampering). Where authentication is enforced, it is frequently Basic auth with weak or default credentials that yield to spraying. Getting past this exposes the write verbs that make WebDAV dangerous.

```bash
# verb tampering: a protected path reachable via an unprotected method
curl -s -X GET http://<target>/protected/        # 401
curl -s -X PROPFIND http://<target>/protected/ -H 'Depth:1' --data ''   # 207 => PROPFIND unguarded
curl -s -X PUT http://<target>/protected/x.txt --data 'test' -i        # does PUT skip auth?
# weak/default Basic auth
curl -s -u admin:admin -X OPTIONS http://<target>/ -i | grep -i allow
hydra -L users.txt -P pass.txt http-get://<target>/protected/
```

## Exploitation notes

- Test each verb against a protected path independently: a 401 on `GET` with a 207 on `PROPFIND` or success on `PUT` reveals method-scoped protection, the classic WebDAV verb-tampering bypass.
- IIS and Apache WebDAV misconfigurations historically allowed exactly this uneven enforcement; always probe `PUT`/`MOVE`/`PROPFIND` even when `GET` is locked.
- Where auth is actually enforced on all verbs, fall back to credential attacks (default/weak Basic auth) or credentials found elsewhere.
- A successful bypass that reaches `PUT`/`MOVE` leads straight to [PUT upload to RCE](put-upload-to-rce.md); one that reaches only `PROPFIND`/`GET` still enables listing and traversal.

## References

- [HackTricks: WebDAV and verb tampering](https://book.hacktricks.xyz/network-services-pentesting/pentesting-web/put-method-webdav)
- [OWASP: testing HTTP methods](https://owasp.org/www-project-web-security-testing-guide/)
