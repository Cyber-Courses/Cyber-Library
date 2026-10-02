---
title: "Expression and script evaluation: OGNL, SpEL, and user-controlled regex sinks"
description: How attacker-influenced strings reach server-side expression engines—OGNL, SpEL, and other object-graph languages—where they execute code, and how user-controlled regular expressions exhaust CPU through catastrophic backtracking.
keywords:
  - expression language injection
  - EL injection
  - OGNL injection
  - SpEL injection
  - ReDoS
  - remote code execution
---

# Expression evaluation

**Expression evaluation** vulnerabilities arise when application code hands an attacker-influenced string to a server-side expression engine that then *evaluates* it instead of treating it as data. Unlike a template engine that renders a file, these sinks sit in business logic: a rule DSL, a `@Value` binding, a framework that evaluates object-graph navigation strings from request parameters, or a regular-expression API compiling a user pattern. The expression language is usually a full member of the host runtime—Java, in the common cases—so a successful injection reaches class loaders and process execution, yielding **remote code execution (RCE)** in the service account's context.

> **Scope.** For authorized penetration tests, red-team engagements, CTF labs, and code review of systems you own or are contracted to assess. Evaluating expressions against systems without written authorization is unlawful.

## Overview

Two distinct sink families are gathered here, and they fail in different ways:

1. **Code-bearing expression languages.** OGNL, SpEL, MVEL, and similar object-graph languages were designed to read and write object properties, call methods, and construct instances. When a value that crosses a trust boundary is parsed and evaluated as one of these expressions, the attacker inherits the language's full reach: `new`, static method calls, reflection, and `Runtime.getRuntime().exec`. These are the engines behind some of the most severe Java web vulnerabilities on record.

2. **Regular-expression evaluation (ReDoS).** A regex engine is a different kind of evaluator. Backtracking engines (PCRE, Java `java.util.regex`, JavaScript, Python `re`) explore an exponential number of match paths on certain pattern/input combinations. When either the **pattern** or the **input** is attacker-controlled, a short string can pin a CPU core for seconds or minutes per request, a denial-of-service primitive that needs no code execution at all.

The decisive question for the first family is: *does this string get parsed as an expression, and is the evaluation full-featured (methods and constructors) or restricted to property reads?* For ReDoS it is: *can the attacker influence the pattern, the subject, or both, and does the engine backtrack?*

## Why it reaches the engine

- **Expressions are a convenient extension point.** Frameworks expose expression evaluation so that non-code configuration—validation rules, access-control conditions, dynamic field mappings—can be expressed as short strings. Developers then feed those strings from a database, a header, or a form without recognizing them as a code sink.
- **Concatenation into a parser.** Building an expression by string concatenation (`parser.parseExpression("user." + field)`) places attacker bytes directly into the grammar, exactly as SQL and shell injection do.
- **Framework internals evaluate silently.** Some stacks evaluate OGNL/SpEL on values the developer never explicitly passed to a parser—error messages, tag attributes, parameter names—so the sink is invisible in application code.
- **Regex from user input.** Search, reporting, and filter features that accept a `regex=` parameter, or that interpolate a user fragment into a larger pattern, turn the matcher into an attacker-tunable workload.

## Impact

For OGNL and SpEL the ceiling is full RCE: file read and write, environment and credential harvesting, internal network access, and a foothold for lateral movement, all bounded only by the service account's privileges and the host's network position. For ReDoS the impact is availability—a single request, or a handful, can saturate worker threads and stall the application—plus its use as an amplifier against rate-limited or asynchronous endpoints where one cheap request buys expensive server work.

## Pages

| Page | Focus |
|------|--------|
| [OGNL and similar object-graph languages](ognl-and-similar-object-graph-languages.md) | OGNL, MVEL and object-graph EL families in Java frameworks: property navigation that escalates to constructor and `exec` payload chains |
| [SpEL and Spring expressions](spel-and-spring-expressions.md) | Spring Expression Language: `T()` type references, `@Value`, and `parseExpression` sinks reaching reflection and runtime execution |
| [ReDoS (user-controlled regex)](redos-user-controlled-regex.md) | Catastrophic backtracking: evil pattern shapes, how user regex or user input triggers worst-case, and amplification |

## References

- [OWASP: Expression Language Injection](https://owasp.org/www-community/vulnerabilities/Expression_Language_Injection)
- [CWE-917: Improper Neutralization of Special Elements used in an Expression Language Statement](https://cwe.mitre.org/data/definitions/917.html)
- [CWE-1333: Inefficient Regular Expression Complexity](https://cwe.mitre.org/data/definitions/1333.html)
- [OWASP: Regular expression Denial of Service (ReDoS)](https://owasp.org/www-community/attacks/Regular_expression_Denial_of_Service_-_ReDoS)
- [PayloadsAllTheThings: Expression Language / OGNL / SpEL](https://github.com/swisskyrepo/PayloadsAllTheThings)
