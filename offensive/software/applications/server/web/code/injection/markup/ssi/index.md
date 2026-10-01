---
title: "SSI injection"
description: "Server-Side Includes expand <!--#...--> directives before a page is served; user input that reaches an SSI-parsed page runs directives for command execution, variable disclosure, and file inclusion."
keywords:
  - SSI injection
  - Server-Side Includes
  - shtml injection
  - exec cmd
  - include virtual
---

# SSI

Server-Side Includes are directives embedded in HTML that the web server expands at response time, before the page reaches the client. They take the form `<!--#directive parameter="value"-->`, which looks like an HTML comment but is parsed and acted on by the server. Apache (`mod_include`), nginx (`ssi on`), and IIS all support the mechanism, classically on files with a `.shtml`, `.shtm`, or `.stm` extension, though any content type can be configured for SSI parsing.

Injection happens when attacker-controlled input is written into a page that the server then parses for SSI directives. If a comment field, filename, log line, or any reflected value lands in an SSI-enabled document and the server evaluates it, an injected `<!--#...-->` directive executes with the privileges of the web server process. The attacker does not need to control the file on disk; the input only has to be present in the byte stream at the moment the SSI parser runs.

The practical reach depends on which directives the server permits. `exec` reaches operating-system commands and is the highest-impact primitive. `echo` leaks server state through built-in and CGI environment variables. `include` reads files and pulls in server-side resources, with path traversal extending its scope. `config`, `set`, and `fsize` round out the directive set and support fingerprinting and chaining.

This subtree covers command execution through `exec`, environment disclosure through `echo`, and file inclusion through `include`.
