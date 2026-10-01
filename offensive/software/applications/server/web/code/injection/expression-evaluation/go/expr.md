---
title: "Expr injection: logic manipulation and host-exposed methods in expr-lang"
description: "The expr-lang language is sandboxed and cannot call arbitrary Go, so an injected expression manipulates the policy or logic decision it drives and can only reach side effects through methods on objects the host put in the environment."
keywords:
  - expr-lang
  - Go expr injection
  - sandboxed expression
  - env methods
  - policy bypass
  - logic manipulation
---

# Expr

`expr` (`github.com/expr-lang/expr`) evaluates an expression against a host-supplied environment and returns a value, typically a boolean for a policy or a number for a computation. It is sandboxed by design: no package imports, no arbitrary Go calls, no filesystem or network primitives in the language itself. An injected `expr` string cannot spawn a process the way a scripting `eval` can. What it can do is decide, with full control, the result the application acts on, and reach any method the host exposed on the objects in the environment.

## The environment is the surface

An `expr` program compiles and runs against an `env`, usually a struct or map. The expression may read those fields, index those maps, and call the methods of those values. That set is exhaustive; there is nothing reachable that the host did not place in scope.

```go
env := map[string]any{
    "user":   user,       // struct with exported fields and methods
    "amount": amount,
}
program, _ := expr.Compile(userExpression, expr.Env(env))
out, _ := expr.Run(program, env)
```

Here the expression can read `user.Role`, `amount`, and call any exported method on `user`. Enumerating those names and methods is the first step on any target.

## Logic and policy manipulation

Where the result gates a decision, controlling the expression controls the decision. This is the most common and most reliable impact.

```
# Force an allow decision
true
user.Role == "admin" or true

# Subvert a numeric rule
amount * 0
amount < 999999999
```

If the evaluated expression is the authorization check, the discount rule, or the alert condition, these flip it directly regardless of the real data.

## Reaching side effects through methods

The only path to behavior beyond computing a value is a method on an environment object that itself performs I/O or execution. `expr` will invoke exported methods on the values in scope, so if the host exposed a type whose methods touch a database, the filesystem, or a command, those methods are callable from the expression with attacker-chosen arguments.

```
# Only reachable if the host exposed such a type/method in env:
user.LoadProfile("../../etc/passwd")
svc.Run("id")
```

Neither is a feature of `expr`; each exists only because the host put that object in `env`. The honest framing for a report is that code execution is not inherent to the language and depends entirely on the exposed environment, so the work is to inventory the methods in scope and test which have side effects.

## Confirming evaluation and resource abuse

Confirm the string is evaluated with arithmetic that is not an echo.

```
6121 * 8
```

A returned `48968` proves evaluation. `expr` also exposes builtins such as `map`, `filter`, and range operators; a large range or a nested comprehension drives CPU and memory work during evaluation, giving a denial-of-service lever when no logic or method impact is available.

```
len(filter(1..10000000, {# > 0}))
```

## References

- [expr-lang documentation](https://expr-lang.org/)
- [expr-lang project](https://github.com/expr-lang/expr)
- [OWASP: Expression Language Injection](https://owasp.org/www-community/vulnerabilities/Expression_Language_Injection)
