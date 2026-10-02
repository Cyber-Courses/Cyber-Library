---
title: "Twig server-side template injection"
description: "Exploiting Twig SSTI: confirming with {{7*7}}, reaching command execution through the filter and map callbacks, and the historical _self registerUndefinedFilterCallback chain."
keywords:
  - Twig SSTI
  - Symfony
  - filter system
  - registerUndefinedFilterCallback
  - _self
---

# Twig

Twig is the default engine in Symfony and is widely used standalone. Confirm injection with `{{7*7}}` returning `49`. Twig has no direct `import` or attribute-walk to PHP the way Python engines reach `os`; instead, RCE comes from filters and functions that hand a string to a PHP callable.

The reliable modern path is the `filter` or `map` filter, which applies a named PHP function to each element. Pointing it at `system` (or `exec`, `passthru`) runs a command:

```twig
{{ ['id'] | filter('system') }}
{{ ['id'] | map('system') | join }}
```

`filter` iterates the array and calls `system('id')` on the element. Where a single value is cleaner, the `reduce` filter also takes an arrow that can call a function, and older code may expose the `sort`/`map` callbacks the same way.

On Twig 1.x, the historical chain used the `_self` object and `registerUndefinedFilterCallback` / `registerUndefinedFunctionCallback` to register `system` or `call_user_func` as the handler for an unknown filter, then invoked it:

```twig
{{ _self.env.registerUndefinedFilterCallback('system') }}{{ _self.env.getFilter('id') }}
{{ _self.env.registerUndefinedFilterCallback('exec') }}{{ _self.env.getFilter('cat /etc/passwd') }}
```

In Twig 2.x and later `_self` returns only the template name, so this chain is closed and the `filter('system')` form is the one to use.

When the application runs Twig through its sandbox extension (`SandboxExtension`), filters and functions are whitelisted and `system` is not callable; `{{7*7}}` still renders but the callbacks are blocked. Test a known-restricted filter to detect the sandbox, and when it is present the injection is limited to information disclosure (reading variables such as Symfony's `app` object) rather than RCE. Symfony's own context frequently exposes `app.request`, environment, and session data even under the sandbox, so enumerate those before concluding the finding is low impact.

## Tools

- tplmap, SSTImap

## References

- Twig documentation: filter, map, sandbox extension
- Symfony security advisories (sandbox bypasses)
- PortSwigger Web Security Academy: Server-side template injection
