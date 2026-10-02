---
title: "Python sandbox escape: recovering builtins and OS access from restricted exec"
description: "Escaping a restricted Python eval/exec: walking the object subclass graph to reach os and subprocess, rebuilding __import__ and __builtins__, and defeating name denylists."
keywords:
  - python sandbox escape
  - __subclasses__
  - __builtins__
  - restricted exec
  - audit hooks
---

# Python sandbox escape

Attempts to run untrusted Python safely, by stripping `__builtins__`, denylisting names like `os`/`eval`/`import`, or filtering source, run into the fact that Python's object model keeps the whole runtime reachable from almost any object. From a confined `eval`/`exec` primitive (a sandboxed template, a "calculator" endpoint, an SSTI landing in a `SandboxedEnvironment`), you walk back to the OS.

## Recover OS access from the object graph

Even with `__builtins__` removed, any object exposes its type hierarchy, and somewhere in the subclass tree sit classes that import or spawn processes:

```python
# From any object, reach every loaded class:
().__class__.__base__.__subclasses__()
```

Index the resulting list to a useful class, for example one of `subprocess.Popen`, `os._wrap_close`, or a warnings/loader class whose globals include `__builtins__`, then call through it:

```python
# Via a subclass whose __globals__ expose builtins/import:
[c for c in ().__class__.__base__.__subclasses__()
   if c.__name__ == 'catch_warnings'][0]()._module.__builtins__['__import__']('os').system('id')
```

A reliable general pattern is to find any function/class with `__globals__` and pull `__builtins__['__import__']` from it, then import `os`/`subprocess`.

## Defeating name denylists

When specific strings are filtered, construct them indirectly:

- `getattr(obj, 'o'+'s')`, or build names from `chr()`/concatenation.
- Reach attributes with encoded names via `getattr`, or through `__getattribute__`.
- Use `__import__` obtained from globals instead of the `import` statement.

## Jinja2 SandboxedEnvironment

The web-specific case is Jinja2's sandbox, where attribute access beginning with `_` is checked by `is_safe_attribute` (so encoding the underscores does not help). The escape is a method the sandbox still allows that leaks the object graph, which is version-dependent; see the [Jinja2 SSTI](../../code/injection/template-engine/python/jinja2.md) page for the template-context specifics. The subclass-walk above is the engine-level primitive underneath it.

## Exploitation notes

- `eval`/`exec` with a restricted `globals` still has access to literals' types, so `().__class__` style bootstraps work even when names are stripped.
- Modern CPython **audit hooks** (`sys.addaudithook`) and real sandboxes (seccomp, separate interpreter, RestrictedPython) can block `os.system`/`subprocess`; check for them and pivot to file read/write or network via whatever module the hooks still permit.
- For output, prefer `subprocess.check_output(...).decode()` returned through the sink, or an OOB callback when nothing is echoed.

## Tools

- Hand-built subclass-walk payloads; SSTImap for template-context automation.

## References

- Python data model: type/subclass introspection
- PortSwigger Web Security Academy: SSTI (sandbox escape)
