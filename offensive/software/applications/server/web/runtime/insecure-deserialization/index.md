---
title: "Insecure deserialization: turning attacker-controlled objects into code execution"
order: 1
description: "How deserialization sinks become RCE: engine callbacks that fire during object reconstruction, gadget chains, and where attacker-controlled serialized data enters, organized by language."
keywords:
  - insecure deserialization
  - object injection
  - gadget chain
  - pickle
  - ysoserial
  - POP chain
---

# Insecure deserialization

Serialization turns an in-memory object into a byte stream; deserialization rebuilds it. The vulnerability is that most engines run **engine-defined callbacks during reconstruction** (magic methods, reducers, readback hooks), so a deserializer handed attacker-controlled bytes does not just produce data, it *executes behavior*. Chaining those callbacks across classes already loaded in the application reaches command execution, file writes, or SSRF. Impact is typically **remote code execution**.

## The shared model

Every language variant follows the same three-part pattern:

1. **A sink** that deserializes untrusted input: `unserialize()`, `pickle.loads()`, `ObjectInputStream.readObject()`, `Marshal.load()`, `BinaryFormatter.Deserialize()`, `node-serialize.unserialize()`.
2. **A trigger**: a callback the engine runs automatically on the rebuilt object (`__wakeup`/`__destruct`, `__reduce__`, `readObject`, `init_with`, an IIFE), or a type the deserializer is told to instantiate.
3. **A gadget chain**: a sequence of methods on classes available in the target's codebase and dependencies that, once entered through the trigger, performs the attacker's action. The chain is assembled from whatever is on the classpath, which is why tools ship curated chains per framework.

## Finding the sink

- Recognize serialized formats on the wire: PHP `O:4:"User":...`, Java base64 beginning `rO0` (hex `AC ED 00 05`), Python pickle opcodes, .NET `AAEAAAD/////`, Ruby Marshal `\x04\x08`.
- Look in cookies, hidden fields, `Authorization`/custom headers, caches, message queues, and import/upload features.
- Confirm by tampering a byte and watching for a deserialization-specific error, then move to a controlled object.

## Languages

- **[PHP object injection](php-object-injection.md)**: `unserialize()`, magic-method POP chains, and reaching the sink without an explicit call via `phar://`.
- **[Python pickle](python-pickle.md)**: `__reduce__` as a direct RCE primitive, plus `yaml.load` and jsonpickle.
- **[Java deserialization](java-deserialization.md)**: `readObject`, ysoserial gadget chains, and common entry points.
- **[Ruby deserialization](ruby-deserialization.md)**: `Marshal.load` and `YAML.load` universal gadget chains.
- **[.NET deserialization](dotnet-deserialization.md)**: `BinaryFormatter`, `TypeNameHandling`, and ViewState.
- **[Node.js deserialization](nodejs-deserialization.md)**: `node-serialize` and function-revival libraries.

## Tools

- **ysoserial** (Java), **ysoserial.net**, **PHPGGC** (PHP gadget chains), **marshalsec** (Java/JVM)

## References

- PortSwigger Web Security Academy: Insecure deserialization
- OWASP: Deserialization Cheat Sheet
