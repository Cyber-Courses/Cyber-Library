---
title: "Discovery and methods: enumerating WebDAV"
description: "Detecting WebDAV on a web server and enumerating its allowed methods with OPTIONS and PROPFIND, and probing which directories accept writes, to map whether PUT, MOVE, and traversal are available before exploiting them."
keywords:
  - WebDAV discovery
  - OPTIONS
  - PROPFIND
  - DAV header
  - davtest
---

# Discovery and methods

The first step is confirming WebDAV and learning what it allows. An OPTIONS request returns the `DAV` header and the `Allow` list of methods, and PROPFIND lists resources. Tools then probe which extensions can be uploaded and executed in each writable directory, which determines whether the server is exploitable through PUT.

```bash
curl -s -X OPTIONS http://<target>/ -i | grep -iE 'DAV|Allow'
curl -s -X PROPFIND http://<target>/ -H 'Depth: 1' --data ''   # list resources
davtest -url http://<target>/                                   # test uploadable/executable types
```

## Exploitation notes

- The `Allow` header reveals whether PUT, MOVE, DELETE, and MKCOL are available, shaping the attack.
- `davtest` reports which file types upload successfully and which then execute, directly flagging the RCE path.
- A directory that accepts PUT is the target for [PUT upload to RCE](put-upload-to-rce.md); MOVE can rename a disallowed extension to an executable one.

## References

- [davtest](https://github.com/cldrn/davtest)
- [HackTricks: pentesting WebDAV](https://book.hacktricks.wiki/en/network-services-pentesting/put-method-webdav.html)
