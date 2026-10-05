---
title: "Authentication bypass: reaching WebDAV past weak auth"
description: "Bypassing WebDAV authentication: default and weak Basic or Digest credentials, directories where authentication is enforced for GET but not for WebDAV methods like PUT, and server-specific flaws that expose the DAV interface without valid credentials."
keywords:
  - WebDAV authentication
  - Basic auth
  - method-based bypass
  - IIS WebDAV
  - unauthorized
---

# Authentication bypass

WebDAV authentication frequently has gaps. Basic and Digest credentials are often weak or default; some configurations enforce authentication on GET but not on the WebDAV write methods, so PUT or MOVE succeed unauthenticated; and server implementations have had flaws that expose the DAV interface regardless of configured auth.

```bash
# Weak/default Basic auth
curl -u admin:admin -X PROPFIND http://<target>/ -H 'Depth: 1'
# Method gap: GET is protected but PUT is not
curl -X PUT http://<target>/test.txt --data 'x' -i
```

## Exploitation notes

- Test whether write methods are protected independently of GET; access-control that only covers read is a common misconfiguration.
- Try vendor defaults and reused credentials from elsewhere in the environment against the DAV realm.
- Unauthenticated PUT leads straight to [PUT upload to RCE](put-upload-to-rce.md).

## References

- [HackTricks: pentesting WebDAV](https://book.hacktricks.wiki/en/network-services-pentesting/put-method-webdav.html)
- [RFC 4918: WebDAV](https://www.rfc-editor.org/rfc/rfc4918)
