---
title: "Flee injection: IL-compiled expressions reaching imported .NET types"
description: "Flee compiles expressions to IL and calls real .NET methods on the types in its Imports and owner context, so an injected expression reaches code execution scoped to the imported type surface."
keywords:
  - Flee
  - Fast Lightweight Expression Evaluator
  - .NET expression injection
  - ExpressionContext Imports
  - IL compilation
  - static method call
---

# Flee

Flee, the Fast Lightweight Expression Evaluator, is not a string interpreter. It compiles each expression to IL and executes it as real .NET code. That compilation is what sets Flee apart from a math-only evaluator: an expression can reference the .NET types that the host made available and call their static methods and properties. When the expression text is attacker-controlled, the reachable impact is whatever those types expose, up to and including code execution.

## What the expression can reach

A Flee expression is evaluated against an `ExpressionContext`. Two parts of that context decide the attack surface:

- **Imports** (`context.Imports`): types and namespaces the host registered so the expression can name them. An imported type is callable by name in the expression.
- **The owner object** (`context.Imports.AddInstance` / the owner passed to the context): its public members are reachable unqualified.

Flee binds to genuine members and emits IL that invokes them. So the question on any target is not whether Flee can call methods (it can) but which types are in scope.

```csharp
var context = new ExpressionContext();
context.Imports.AddType(typeof(Math));
var e = context.CompileGeneric<double>(userExpression);
double result = e.Evaluate();
```

With only `Math` imported, `userExpression` reaches `Math.Sqrt`, `Math.Pow`, and the arithmetic grammar. The surface is still just that type.

## Escalation depends on the imported surface

The moment the host imports a type that exposes a dangerous method, an injected expression can call it. The canonical example is a context that has `System.Diagnostics.Process` in scope, which turns the evaluator into a process launcher.

```
# With System.Diagnostics.Process imported:
Process.Start("cmd.exe", "/c whoami")

# With System.IO.File imported:
File.ReadAllText("C:\\inetpub\\wwwroot\\web.config")
```

Even without an obviously dangerous import, a type brought in for convenience can be a pivot: anything exposing a static property that returns a richer object, a factory, or a method that touches the filesystem or process table extends reach. Enumerate the imports first, because they are the exact boundary of what the injection can do.

## Confirming compilation and probing scope

Start by proving the input is compiled as a Flee expression, with arithmetic that cannot be a coincidental echo.

```
8123477 * 3
```

A returned `24370431` confirms evaluation. Next, probe which types are in scope by naming them and reading a harmless static member; a successful compile means the type is imported and its members are callable.

```
# Does the context import these? A clean evaluate answers yes.
DateTime.Now
Environment.MachineName
```

Once a reachable type with a useful method is confirmed, call it. If nothing dangerous is imported, Flee is still an arithmetic and logic surface: flip a boolean rule, skew a computed price, or drive an expensive computation for resource abuse. The step up to code execution is gated entirely by the import list, so report the impact against the types actually in context rather than assuming the worst.

## Tools

- Manual testing with Burp Repeater; payloads crafted per engine.
- **Burp Intruder**: probe which .NET types are imported into the ExpressionContext.

## References

- [Flee project (Fast Lightweight Expression Evaluator)](https://github.com/mparlak/Flee)
- [OWASP: Expression Language Injection](https://owasp.org/www-community/vulnerabilities/Expression_Language_Injection)
