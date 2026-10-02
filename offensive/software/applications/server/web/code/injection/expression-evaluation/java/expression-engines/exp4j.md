---
title: "Exp4j injection: abusing a sandboxed mathematical evaluator"
description: "Exp4j parses numbers, operators, and host-registered functions only, so untrusted input into an Exp4j expression reaches the math grammar, exposed custom functions and variables, and resource exhaustion, not the JVM runtime."
keywords:
  - Exp4j
  - Exp4j injection
  - expression evaluator
  - custom function abuse
  - denial of service
---

# Exp4j

Exp4j is a small mathematical expression evaluator for the JVM. It parses a string into an abstract syntax tree of numbers, operators, parentheses, a fixed set of built-in functions (`sin`, `cos`, `log`, `sqrt`, `abs`, and so on), and whatever variables and custom functions the host application registered through the `ExpressionBuilder`. It evaluates that tree to a single `double`.

That is the whole language. Exp4j has no notion of Java types, no reflection, no object construction, and no access to the runtime. An expression cannot name a class, cannot call a method on an object, and cannot reach `Runtime` or `ProcessBuilder`. Injecting into an Exp4j expression does not give code execution, and a payload like `Runtime.getRuntime().exec(...)` simply fails to parse. Framing Exp4j as an RCE sink is wrong, and an assessment that claims it is will not survive validation.

What injection into an Exp4j expression actually gives is control of the computation, bounded by what the host exposed.

## Vulnerable pattern

```java
// formula taken from the request
Expression e = new ExpressionBuilder(formula)
    .variables("x", "y")
    .build()
    .setVariable("x", xValue)
    .setVariable("y", yValue);
double result = e.evaluate();
```

The application intends the user to supply something like `x * 1.2 + y`. Because the raw string is handed to `ExpressionBuilder`, the user instead supplies any expression the grammar accepts.

## Exposed variables and custom functions

The reachable surface is defined by the context the host built. Every variable registered with `variables(...)` or `setVariable(...)` is readable, so an injected formula can pull values the interface never meant to combine or return, for example reflecting a sensitive registered variable straight back as the result:

```
discount_rate
```

Custom functions registered with `.function(new Function("name", argc){...})` are callable by name. Exp4j itself cannot do anything dangerous, but if the application registered a custom function that does something meaningful (a lookup, a currency conversion that hits a backend, a function that reads a configured value), that function is reachable from the injected expression with attacker-chosen arguments:

```
lookup(99999) + rate(payload)
```

This is the one case where Exp4j injection reaches beyond arithmetic, and it reaches exactly as far as the registered function goes and no further. Enumerating which function and variable names resolve, versus which raise an unknown-function or unknown-variable error, maps the available context before crafting a payload.

## Resource exhaustion

Because evaluation is pure arithmetic over `double`, the abuse that does not depend on the host context is making that arithmetic expensive or degenerate. Large exponentiation and deeply chained operations force heavy computation per request:

```
9^9^9^9
```

Deeply nested parentheses and very long operator chains push parsing and tree construction, and expressions that drive results to `Infinity` or `NaN` (division by zero, `log(0)`, massive factorials where exposed) can propagate a surprising value into whatever the application does with the returned `double` downstream. None of this escapes the evaluator; the impact is availability and corrupted computed values, not execution.

## Tools

- Manual testing with Burp Repeater; payloads crafted per engine.
- **Burp Intruder**: enumerate registered variables and custom functions by name.

## References

- [Exp4j documentation](https://www.objecthunter.net/exp4j/)
- [Exp4j custom functions and operators](https://www.objecthunter.net/exp4j/#Custom_functions)
