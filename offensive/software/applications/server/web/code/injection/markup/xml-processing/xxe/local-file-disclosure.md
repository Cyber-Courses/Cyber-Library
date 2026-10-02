---
title: "Local file disclosure via in-band XXE"
description: "A SYSTEM entity reads a local file and the application reflects the expanded value back in its response, returning file contents directly."
keywords:
  - XXE file read
  - SYSTEM entity
  - file disclosure
  - php filter base64
  - in-band XXE
---

# Local file disclosure

The classic XXE is in-band: declare an external **general entity** that points at a local file, reference it somewhere the application echoes back, and read the file straight out of the HTTP response. This works whenever the parser resolves external entities and some parsed value is reflected to the client, for example an error message, a search result, or a field rendered back into a confirmation page.

## In-band read

Define the entity in the internal subset (the bracketed DTD) and reference it in an element whose text the application returns:

```xml
<?xml version="1.0"?>
<!DOCTYPE foo [
  <!ENTITY xxe SYSTEM "file:///etc/passwd">
]>
<stockCheck><productId>&xxe;</productId></stockCheck>
```

When the parser expands `&xxe;`, the `productId` value becomes the contents of `/etc/passwd`, and the application reflects it back. The `file://` scheme reads any path the service account can open: `file:///etc/passwd`, `file:///etc/hostname`, `file:///proc/self/environ`, `file:///home/user/.ssh/id_rsa`, or a Windows path such as `file:///c:/windows/win.ini`.

To find the injectable field, probe with an **external** entity, not an internal one. An internal replacement entity such as `<!ENTITY test "INJECTED">` expands even when external entity loading is disabled, so seeing it echoed proves only that entities are processed, not that `file://` reads work, a false positive on a hardened parser. Point the probe at a file that reliably exists instead:

```xml
<!DOCTYPE foo [ <!ENTITY test SYSTEM "file:///etc/hostname"> ]>
<stockCheck><productId>&test;</productId></stockCheck>
```

If the hostname comes back in the response, the parser is resolving external `file://` entities through that field and the full read above will work.

## Files that break the parser

Many interesting files are not well-formed character data. XML forbids raw `<`, `&`, and certain control bytes in content, so reading a source file, a config with `<` characters, or any binary blob makes the parser abort with a fatal error before it returns anything. On PHP targets the `php://filter` wrapper solves this by transforming the resource before it reaches the parser:

```xml
<!DOCTYPE foo [
  <!ENTITY xxe SYSTEM
    "php://filter/convert.base64-encode/resource=/var/www/html/config.php">
]>
<stockCheck><productId>&xxe;</productId></stockCheck>
```

The file is base64-encoded, so it contains only `[A-Za-z0-9+/=]` and always parses cleanly. Decode the reflected string to recover the original bytes, which makes this the standard way to pull PHP source, database credentials, and binary files through an in-band channel. Chain filters to handle awkward encodings, for example `php://filter/read=convert.base64-encode/resource=index.php`.

Other schemes extend reach depending on the platform and installed extensions: `expect://id` for command execution where the expect wrapper is loaded, `data://` for inlining a crafted payload, and `netdoc://` on some Java stacks as an alternative file reader. When the file contents are not reflected at all, switch to an out-of-band or error-based channel.

## Tools

- **XXEinjector**: automating in-band file retrieval through XXE.
- **Burp Suite**: crafting and iterating SYSTEM entity payloads in Repeater.
- Manual testing with `php://filter` base64 payloads for non-XML files.

## References

- [OWASP: XML External Entity (XXE) Processing](https://owasp.org/www-community/vulnerabilities/XML_External_Entity_(XXE)_Processing)
- [PayloadsAllTheThings: XXE Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/XXE%20Injection)
