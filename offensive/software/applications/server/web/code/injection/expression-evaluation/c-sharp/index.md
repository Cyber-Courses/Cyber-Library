---
title: "C# expression evaluation injection"
description: ".NET expression and formula evaluators, from math-only engines like NCalc to the IL-compiling Flee, plus ASP.NET data-binding evaluation that reaches the page runtime."
keywords:
  - C# expression injection
  - .NET expression evaluator
  - NCalc
  - Flee
  - ASP.NET data binding
  - DataBinder.Eval
---

# C#

.NET applications reach for an expression evaluator whenever a formula, rule, or price needs to be configurable without a redeploy. Those evaluators span a wide impact range. A math-and-logic engine confined to numbers and booleans is a different target from one that compiles to IL and can call real .NET methods, and a data-binding expression that runs in the page context is different again.

The distinction that decides outcome is what the engine can reach. NCalc parses arithmetic, comparisons, and whatever functions the host registered, and nothing more by itself. Flee compiles each expression to IL and invokes genuine .NET methods on the types the host imported, so its ceiling is the imported type surface. ASP.NET data binding evaluates a property path as a .NET expression in the page, so attacker control of the expression string (not merely the bound value) runs in that page's context.

## Engines

- **[ASP.NET data binding](asp-net-data-binding.md)**: `Eval`/`DataBinder.Eval` expressions evaluated against the page, where a controlled expression string runs as a .NET expression.
- **[Flee](flee.md)**: the Fast Lightweight Expression Evaluator, which compiles to IL and can reach static methods and properties on imported types.
- **[NCalc](ncalc.md)**: a mathematical and logical evaluator whose injectable surface is the math/logic grammar, the parameter and function host hooks, and any custom functions the application registered.

Each page maps the engine's parse and evaluation entry point, what its grammar reaches, and the realistic escalation from a benign-looking formula to the engine's maximum impact.

## Tools

- **Burp Suite**: Repeater and Intruder for injecting formula and data-binding payloads.
- Manual testing with Burp Repeater; payloads crafted per .NET engine.

## References

- [OWASP: Expression Language Injection](https://owasp.org/www-community/vulnerabilities/Expression_Language_Injection)
- [OWASP WSTG: Testing for Code Injection](https://owasp.org/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/07-Input_Validation_Testing/11-Testing_for_Code_Injection)
