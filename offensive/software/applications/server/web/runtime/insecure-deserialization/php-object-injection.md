---
title: "PHP object injection: unserialize() magic-method POP chains"
order: 1
description: "Exploiting PHP unserialize() on attacker input: crafting serialized objects, driving magic methods (__wakeup, __destruct, __toString) into property-oriented programming chains, and reaching the sink via phar."
keywords:
  - PHP object injection
  - unserialize
  - POP chain
  - __wakeup
  - __destruct
  - PHPGGC
---

# PHP object injection

PHP object injection occurs when `unserialize()` is called on attacker-controlled data. PHP rebuilds the object graph described by the string, including its class and property values, and runs **magic methods** during and after reconstruction. An attacker who controls the serialized string controls which classes are instantiated and with which properties, then rides the magic methods into a **property-oriented programming (POP) chain**.

## The serialized format

PHP serialization is human-readable and craftable by hand:

```
O:4:"User":2:{s:4:"name";s:5:"admin";s:7:"isAdmin";b:1;}
```

`O:4:"User"` is an object of a 4-character class name `User` with 2 properties. Types are explicit (`s` string, `i` int, `b` bool, `a` array, `O` object), so you can forge any object of any class the application can autoload, with any property values, including nested objects.

## The triggers

PHP runs these magic methods on a rebuilt object without the application calling them:

- `__wakeup()`: runs immediately on `unserialize()`.
- `__destruct()`: runs when the object is garbage-collected at end of request.
- `__toString()`: runs when the object is used in string context.
- `__call()`, `__get()`, `__set()`: run on access to missing methods/properties.

Any of these on a loadable class is an entry point. The attack does not need the class to be obviously dangerous; it needs a method that, when it runs with attacker-chosen properties, does something useful, or that calls another object's method, extending the chain.

## POP chains

A POP chain strings together methods across classes that already exist in the application and its dependencies. A representative shape: an entry object's `__destruct()` calls `$this->handler->close()`; the attacker sets `handler` to an object whose `close()` writes `$this->data` to `$this->path`; the attacker sets `path` to a webroot file and `data` to a PHP payload. Reconstruction plus `__destruct` becomes an arbitrary file write.

Because chains depend on the exact libraries present, use **PHPGGC** to generate payloads for known frameworks (Laravel, Symfony, Monolog, Guzzle, Doctrine, WordPress):

```bash
phpggc Monolog/RCE1 system 'id'            # print a payload
phpggc -b Monolog/RCE1 system 'id'         # base64-encoded
phpggc Guzzle/RCE1 system 'id'
```

Feed the output to the sink (cookie, parameter, header) exactly as the application expects it.

## Reaching unserialize without an explicit call

A sink does not require a literal `unserialize()` on your input. PHP deserializes a Phar archive's metadata when a file operation touches a `phar://` path, so file functions acting on an attacker-influenced path become deserialization sinks. See [phar deserialization](../file-wrappers-and-stream-handlers/phar-deserialization.md) for building the archive and triggering it.

## Exploitation notes

- `__wakeup()` was historically skippable (an object-count mismatch bug) to defeat guards; on modern PHP rely on `__destruct()` and the other magic methods instead.
- Private and protected property names serialize with null-byte prefixes (`\0ClassName\0prop` / `\0*\0prop`); preserve them exactly when crafting by hand.
- Enumerate loadable classes and their magic methods from source or leaked paths; the chain is only as good as the gadgets on disk.

## Tools

- **PHPGGC**: curated gadget chains for common frameworks.

## References

- PortSwigger Web Security Academy: Insecure deserialization (PHP)
- PHP manual: Object serialization, magic methods
