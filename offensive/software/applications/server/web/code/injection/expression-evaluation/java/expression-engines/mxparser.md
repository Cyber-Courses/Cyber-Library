---
title: "mXparser injection: abusing a scientific math parser"
description: "mXparser evaluates mathematical expressions with user-defined functions, arguments, and constants but has no access to Java types or the runtime, so injection reaches the math surface, exposed user-defined elements, and resource abuse rather than code execution."
keywords:
  - mXparser
  - mXparser injection
  - math expression parser
  - user-defined function
  - denial of service
---

# mXparser

mXparser is a feature-rich mathematical expression parser for the JVM and .NET. It handles a large library of built-in functions (trigonometric, hyperbolic, special, probability, combinatorial), operators, iterated and summation operators, physical and mathematical constants, and user-defined elements: `Argument`, `Constant`, and `Function` objects the host registers on the `Expression` before parsing.

Despite the breadth, mXparser is still a mathematics engine. It evaluates to a numeric result and has no mechanism to reference Java classes, construct objects, invoke methods, or reach reflection, `Runtime`, or `ProcessBuilder`. An injected mXparser expression therefore does not produce code execution, and shell-command payloads are not valid syntax in the grammar. Describe the impact honestly: the vector is the mathematical surface and the host-defined elements, not the JVM.

## Vulnerable pattern

```java
// formula supplied in the request
Expression ex = new Expression(formula);
ex.addArguments(new Argument("x", xValue));
double result = ex.calculate();
```

The intended input is an arithmetic formula in `x`. The raw string reaches the parser, so an attacker supplies any expression mXparser accepts.

## Exposed user-defined elements

The reachable context is whatever the application registered. User-defined `Argument`s and `Constant`s are readable by name, so an injected expression can surface a value the interface never intended to expose by naming it directly. User-defined `Function`s added with `addFunctions(...)` are callable with attacker-chosen arguments:

```
pricing(0) + secretMargin
```

As with any math engine, mXparser does nothing dangerous on its own; the registered function reaches exactly as far as the host implemented it. Probing which identifiers evaluate versus which leave the expression in an error state (mXparser returns `NaN` and exposes `getErrorMessage()`) enumerates the available arguments, constants, and functions.

mXparser also parses expressions that build up its own user-defined elements inline, including the function-definition and iteration syntax, which widens the grammar an injected string can exercise but keeps it inside the numeric model.

## Resource exhaustion

The context-independent abuse is forcing expensive evaluation. Iterated operators over large ranges do substantial work per request, for example a summation across a huge index span:

```
sum(i, 1, 100000000, i^2)
```

Deeply nested functions, large factorials and gamma evaluations, and expressions engineered to return `NaN` or infinite values push computation cost and can propagate degenerate numbers into downstream application logic. The impact is availability and corrupted results, contained within the evaluator.

## References

- [mXparser documentation](https://mathparser.org/)
- [mXparser user-defined functions and arguments](https://mathparser.org/mxparser-tutorial/)
