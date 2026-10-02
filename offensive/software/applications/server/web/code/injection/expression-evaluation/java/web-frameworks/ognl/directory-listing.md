---
title: "OGNL directory listing: confirming Struts2 expression evaluation blind"
description: "A Struts2 OGNL evaluation probe uses arithmetic and string operations inside %{...} to prove the expression language runs, marking the response without touching the member-access sandbox."
keywords:
  - OGNL evaluation probe
  - Struts2 OGNL detection
  - OGNL arithmetic marker
  - blind OGNL injection
  - value stack
---

# Directory listing

Before any runtime payload, the question is whether the injection point reaches OGNL evaluation at all. A direct `Runtime` call is blocked by the member-access sandbox and may also be filtered, so a confirmation probe avoids both: it uses only arithmetic and string operations, which the sandbox never restricts, and watches for the evaluated result in the response. This is the OGNL analogue of a directory-listing or marker check, listing what the value stack resolves rather than invoking the host runtime.

## Arithmetic marker

The cleanest signal is arithmetic the server computes. If the parameter is reflected and OGNL runs, the response carries the product rather than the literal:

```
%{233*233}
```

A response containing `54289` proves evaluation. The same idea seeds a named variable and reads it back, which survives contexts where a bare number is coerced away:

```
%{#probe=233*233}
```

## String-evaluation marker

A string transform is harder to confuse with incidental numbers in the page. The J2EEScan-style probe builds a known token through `replace`, so the marker only appears if OGNL evaluated the call:

```
${"zkz".replace("z","s")}
```

A response containing `sks` confirms both that OGNL ran and that method calls on core `String` are reachable. A multiplication-plus-concatenation variant combines both signals into one unlikely token:

```
%{"res_"+(233*233)}
```

## Enumerating the context

Once evaluation is confirmed, the same sandbox-safe grammar reads the stack to list what is reachable, which guides the escalation payload:

```
%{#context}
%{#context.keySet()}
%{@java.lang.System@getProperty("user.dir")}
```

`#context` enumerates the OGNL context keys, including the member-access entry that the [remote code execution](remote-code-execution.md) page clears, and `user.dir` returns the working directory that frames later file paths. Reaching `@java.lang.System@getProperty` without an error already indicates a relaxed or already-cleared sandbox on that version.

## Tools

- **Burp Suite**: Repeater to send arithmetic and string-replace evaluation probes.
- **J2EEScan**: Burp extension whose OGNL probe uses the "zkz".replace marker.

## References

- [Apache Commons OGNL Language Guide](https://commons.apache.org/proper/commons-ognl/language-guide.html)
- [Apache Struts2 Security](https://struts.apache.org/security/)
