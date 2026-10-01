---
title: "ASP.NET data binding injection: Eval and DataBinder.Eval expression control"
description: "When attacker input reaches the expression string of a data-binding tag such as Eval or DataBinder.Eval, it evaluates as a .NET expression in the page context and reaches code execution."
keywords:
  - ASP.NET data binding
  - DataBinder.Eval
  - Eval expression injection
  - data-binding expression
  - Web Forms
  - .NET code execution
---

# ASP.NET data binding

ASP.NET Web Forms data-binding expressions, written `<%# ... %>`, are evaluated when a control's `DataBind()` runs. The classic form is `<%# Eval("PropertyName") %>`, which resolves a property path against the current data item, and `DataBinder.Eval(Container.DataItem, "PropertyName")`, which does the same programmatically. These are not string substitutions. The content between `<%#` and `%>` is a .NET expression compiled into the page class, so whatever it contains runs with the full capability of page code.

## The condition that matters

The dangerous case is narrow and specific: the attacker has to control the expression **text**, the content between `<%#` and `%>`, not merely a value bound into a property, and not merely the string argument handed to `Eval`. Controlling only the `Eval` argument is weak: `Eval(attackerString)` still treats that string as a property/indexer navigation path against the data item, so the most it yields is reading other properties of the bound object graph, not code execution. Code execution appears only when the application assembles the expression text itself from input and then compiles it.

```
# Benign: the value is data, the expression is fixed
<%# Eval("DisplayName") %>

# Weak: only the Eval ARGUMENT is attacker-controlled -> property-path
# navigation of the data item, not compiled code
<%# Eval(userSuppliedPath) %>

# Dangerous: the attacker controls the EXPRESSION TEXT that gets compiled
<%# {attacker-controlled .NET expression} %>
```

When the attacker-writable store feeds the expression text (a template field, a saved report definition, persisted markup, a runtime-composed `.aspx` fragment), that text is parsed and compiled as a .NET expression against the page, which is the code-execution condition.

## Reaching the sink

Three patterns put attacker-controlled text into an evaluated expression:

- A reporting or dashboard feature that lets users define a column as an expression, stored and later emitted into a `<%# ... %>` binding or passed to `DataBinder.Eval`.
- A page-template or theme system that persists markup containing binding expressions, where the persisted markup is attacker-writable.
- Dynamic page generation that concatenates input into an `.aspx` fragment compiled at runtime.

Because the expression compiles as page code, it is not confined to property resolution. A controlled expression can instantiate types and call methods reachable from the page's namespace imports.

```
<%# System.Diagnostics.Process.Start("cmd.exe","/c whoami") %>
<%# new System.IO.StreamReader("c:\\inetpub\\wwwroot\\web.config").ReadToEnd() %>
```

The first reaches process start; the second reads a file from the application context. Both run under the application pool identity, which is the ceiling of impact here.

## Confirming evaluation

Prove the expression is compiled rather than echoed with an arithmetic or string probe whose result differs from its source text.

```
<%# (7777000 * 7) %>
<%# "aa" + "bb" %>
```

A response containing `54439000` or `aabb` confirms the fragment was evaluated as an expression, not emitted literally. From there, move to type instantiation and method calls as above, scoping payloads to the namespaces the page imports (`System`, and whatever the application adds).

## References

- [Microsoft: Data-binding expression syntax](https://learn.microsoft.com/en-us/previous-versions/aspnet/ms178366(v=vs.100))
- [Microsoft: DataBinder.Eval method](https://learn.microsoft.com/en-us/dotnet/api/system.web.ui.databinder.eval)
- [OWASP: Expression Language Injection](https://owasp.org/www-community/vulnerabilities/Expression_Language_Injection)
