---
title: "C# server-side template injection"
description: "SSTI in .NET template engines: Razor (RazorEngine and runtime-compiled views) compiling embedded C# to reach System.Diagnostics.Process."
keywords:
  - C# SSTI
  - .NET template injection
  - Razor
  - RazorEngine
---

# C#

.NET template injection centers on Razor, the view engine behind ASP.NET and the standalone RazorEngine / RazorLight libraries. Razor compiles its templates to C#, so where untrusted input is compiled as a Razor template, the injection runs arbitrary C# and reaches `System.Diagnostics.Process.Start`, giving direct command execution with no sandbox.

Razor uses `@` for code: `@(7*7)` renders `49`, and `@{ ... }` blocks run statements.

## Engines

- **[Razor](razor.md)**: `@` expressions and code blocks compiling to C#, the RazorEngine/RazorLight `RunCompile` sink, and `Process.Start` for RCE.

## References

- Microsoft Razor documentation; RazorEngine / RazorLight projects
- PortSwigger Web Security Academy: Server-side template injection
