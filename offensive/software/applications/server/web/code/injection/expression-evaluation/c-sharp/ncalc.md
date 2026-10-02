---
title: "NCalc injection: abusing a .NET math and logic evaluator"
description: "NCalc evaluates arithmetic, logic, and registered functions but provides no reflection or arbitrary code execution on its own, so the injectable surface is the grammar, the EvaluateParameter and EvaluateFunction hooks, and any dangerous custom function."
keywords:
  - NCalc
  - .NET math evaluator
  - EvaluateFunction
  - EvaluateParameter
  - custom function RCE
  - logic manipulation
---

# NCalc

NCalc is a mathematical and logical expression evaluator for .NET. It parses arithmetic, comparisons, boolean logic, and a set of built-in and host-registered functions, and it returns the computed value. It is important to be precise about what standard NCalc does not do: by itself it offers no reflection, no type references, and no arbitrary .NET execution. An injected NCalc expression cannot reach `Process.Start` the way a compiled evaluator can. The real attack surface is the grammar it does expose, the host hooks around it, and whatever functions the application chose to register.

## The grammar as a surface

NCalc evaluates the full arithmetic and logical grammar, including comparisons, conditionals, and the built-in function set (`if`, `in`, `Abs`, `Pow`, string helpers, and similar). That grammar is enough to subvert any decision the application derives from the result.

```
# Force a rule or authorization branch
1 == 1
true or false

# Skew a computed value
basePrice * 0
if(true, 0, basePrice)
```

Where the evaluated result feeds a price, a discount, a threshold, or an allow/deny decision, controlling the expression controls that outcome directly. This is logic manipulation rather than code execution, and it is frequently the whole impact on a given target.

## The host hooks

NCalc raises two events the host wires up: `EvaluateParameter`, which resolves named parameters, and `EvaluateFunction`, which resolves function calls the engine does not recognize. These handlers are application code, and an injected expression chooses which parameters and functions to invoke.

```csharp
var e = new Expression(userInput);
e.EvaluateFunction += (name, args) => {
    if (name == "lookup") args.Result = Db.Lookup((string)args.Parameters[0].Evaluate());
};
```

An expression can call `lookup(...)` with attacker-chosen arguments, pass values the developer never intended, and chain registered functions. The reachable behavior is exactly the set of parameters and functions the host resolves, so enumerate them: every custom function name that evaluates cleanly is callable with controlled arguments.

## The only code-execution path

NCalc reaches code execution only when the application has registered a custom function that itself does something dangerous. A function that shells out, reads files, queries a database, or reflects over types becomes the payload, and NCalc is merely the way to call it with attacker input.

```
# Only if the host registered such a function:
exec('whoami')
readfile('C:\\inetpub\\wwwroot\\web.config')
```

Neither exists in stock NCalc. Their presence is a property of the target, so the assessment is to discover the registered function set and test each for side effects, rather than to assume a generic NCalc RCE that does not exist.

## Confirming evaluation and resource abuse

Prove the input is evaluated with arithmetic that is not a coincidental echo.

```
9134 * 7
```

A returned `63938` confirms the string is parsed as an NCalc expression. Beyond logic and function abuse, the grammar also supports resource abuse: a deeply nested or large arithmetic expression forces CPU and memory work during parse and evaluation, giving a denial-of-service lever where no stronger impact is reachable.

## Tools

- Manual testing with Burp Repeater; payloads crafted per engine.
- **Burp Intruder**: enumerate registered parameters and custom functions by name.

## References

- [NCalc documentation](https://ncalc.github.io/ncalc/)
- [NCalc project](https://github.com/ncalc/ncalc)
- [OWASP: Expression Language Injection](https://owasp.org/www-community/vulnerabilities/Expression_Language_Injection)
