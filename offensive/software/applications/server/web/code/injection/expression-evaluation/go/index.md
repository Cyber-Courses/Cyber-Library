---
title: "Go expression evaluation injection"
description: "Go applications embed sandboxed expression mini-languages like expr, where injection manipulates logic and policy decisions and the only path to dangerous behavior is a method on a host-exposed object."
keywords:
  - Go expression injection
  - expr-lang
  - sandboxed expression language
  - policy evaluation
  - environment methods
  - logic manipulation
---

# Go

Go services embed an expression language when a rule, filter, or policy needs to be configurable at runtime: feature flags, pricing rules, alerting conditions, and access policies are common. The dominant library is `expr` (`github.com/expr-lang/expr`), a deliberately sandboxed, non-Turing-complete language. Unlike a JVM EL or a scripting `eval`, it cannot import packages or call arbitrary Go, so there is no inherent code-execution primitive.

That design shapes the whole attack model. When the expression string is attacker-controlled, the payoff is subverting the decision the expression drives, and the only route to anything heavier is calling a method on an object the host placed in the evaluation environment. If that environment exposes a type whose methods perform I/O or execution, those methods are reachable; if it does not, the ceiling is logic and policy manipulation plus resource abuse.

## Engines

- **[Expr](expr.md)**: the `expr-lang/expr` language, its environment model, and how injection reaches logic manipulation and host-exposed methods.

## What the environment decides

Everything reachable in an `expr` expression comes from the `env` the host passes at compile and run time. The grammar can read those variables, index those maps, and call the methods those objects expose, and nothing else. The assessment of any Go expression target is therefore an enumeration of the environment: which variables, which types, and which of their methods are in scope. That boundary is the entire surface.

## Tools

- Manual testing with Burp Repeater; payloads crafted per engine.
- **Burp Intruder**: enumerate the variables, types, and methods exposed in the env.

## References

- [expr-lang documentation](https://expr-lang.org/)
- [OWASP: Expression Language Injection](https://owasp.org/www-community/vulnerabilities/Expression_Language_Injection)
