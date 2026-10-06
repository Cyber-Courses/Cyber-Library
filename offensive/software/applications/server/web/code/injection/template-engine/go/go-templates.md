---
title: "Go text/template and html/template injection"
description: "Exploiting Go template injection: confirming with {{7*7}} behavior, enumerating fields and calling methods on the data object, reading the data model, and the limits that usually prevent RCE."
keywords:
  - Go template injection
  - text/template
  - html/template
  - call built-in
  - method invocation
---

# Templates

Go templates use `{{ }}`, but `{{7*7}}` does not render `49`: arithmetic is not expression syntax in Go templates, and `{{7*7}}` produces a parse error. The reliable probes are `{{.}}`, which prints the entire data object passed to the template, and `{{printf "%d" (len "aaa")}}` style calls using the built-in functions. A parse error mentioning `template:` on malformed input also fingerprints the engine.

What the template can do is bounded by design. It can:

- Print the data object and walk its exported fields: `{{.}}`, `{{.SomeField}}`, `{{.User.Email}}`.
- Call exported methods on the data (and on their results), with arguments: `{{.SomeMethod "arg"}}`.
- Use the built-in functions (`print`, `printf`, `call`, `index`, `len`, `slice`, and so on).

It cannot import packages, instantiate types, or reach `os`/`exec` on its own. So the first impact is information disclosure: dumping `{{.}}` frequently leaks the whole context struct, which may include secrets, tokens, internal configuration, or request data the developer did not intend to expose.

Escalation to code execution depends entirely on the methods reachable from the data object. If any exported method in the reachable object graph runs a command, writes a file, or performs another sensitive action, the template can call it. The `call` built-in invokes a function value passed into the data, so a context that exposes a function field widens what is reachable:

```gotemplate
{{ call .SomeFunc "id" }}
{{ .Cmd.Run }}
```

The practical workflow is: confirm with `{{.}}`, enumerate exported fields and methods, read anything sensitive in the dumped context, then look for a reachable method with a side effect. Treat Go SSTI as information disclosure by default and escalate only when the application's own types hand the template a dangerous method. `html/template` adds contextual output escaping (so reflected values are HTML-encoded) but does not change what the template language can reach, so the enumeration and method-call analysis are identical.

## Tools

- Manual enumeration; tplmap has limited Go support

## References

- Go documentation: text/template (actions, functions), html/template
- PortSwigger Web Security Academy: Server-side template injection
