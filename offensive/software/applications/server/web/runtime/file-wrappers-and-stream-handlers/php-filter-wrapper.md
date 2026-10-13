---
title: "php://filter wrapper: source disclosure and content transformation"
order: 1
description: "Using php://filter to read PHP source and arbitrary files through conversion filters such as convert.base64-encode, defeating the engine's execution of included files."
keywords:
  - php://filter
  - convert.base64-encode
  - source disclosure
  - LFI
  - conversion filter
---

# php filter wrapper

The `php://filter` wrapper applies one or more stream filters to a resource as it is read. Its primary offensive use is **source disclosure**: when an application includes or reads a path you influence, wrapping the target in a base64-encode filter returns the file's bytes instead of executing them, so you recover PHP source that would otherwise just run.

## Reading source

An `include`/`require` normally executes PHP, so a plain LFI of a `.php` file yields its output, not its code. Encode it first:

```
php://filter/convert.base64-encode/resource=config.php
php://filter/convert.base64-encode/resource=../../../../etc/passwd
```

Fed to the sink (for example `?page=php://filter/convert.base64-encode/resource=config`), the include returns base64 of the file; decode it offline. Because the content is base64, it survives being passed through the PHP engine without being interpreted, which is the whole point for `.php` targets.

## Useful filter stacks

Filters can be chained and come in several families:

- **convert.base64-encode / convert.base64-decode**: the workhorse for safe extraction and for decoding.
- **convert.iconv.\***: character-set conversions (the basis of the RCE technique in [filter chains to RCE](filter-chains-to-rce.md)).
- **zlib.inflate / zlib.deflate**: (de)compression, useful to reach compressed content or as chain steps.
- **string.rot13 / string.toupper**: simple transforms, occasionally handy to dodge naive filters on the input.

```
php://filter/read=convert.base64-encode/resource=index.php
php://filter/zlib.inflate/convert.base64-encode/resource=app.gz
```

## Exploitation notes

- Works against sinks like `include`, `require`, `file_get_contents`, `readfile`, `fopen`, and `highlight_file` where the path is attacker-influenced; it does not require `allow_url_include`.
- If the application appends an extension (`$page . ".php"`), target files accordingly or use path traversal plus the wrapper together.
- Source you recover feeds every other attack: it reveals secrets, the exact deserialization sinks, and the classes available for POP chains.
- When you need code execution rather than disclosure, escalate with [filter chains to RCE](filter-chains-to-rce.md), [phar deserialization](phar-deserialization.md), or the [data and input wrappers](data-and-input-wrappers.md).

## Tools

- **curl** / Burp Repeater; base64 decoding offline.

## References

- PHP manual: php://filter, stream filters
- PortSwigger Web Security Academy: LFI and PHP wrappers
