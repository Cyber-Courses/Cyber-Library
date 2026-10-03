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

## Pages

- **[Command execution](command-execution.md)**: The exec directive runs operating-system commands when attacker input reaches an SSI-parsed page, turning reflected markup into remote code execution as the...
- **[Environment disclosure](environment-disclosure.md)**: The echo directive prints SSI and CGI environment variables into the page, leaking paths, client data, and server configuration when input reaches an SSI-par...
- **[File inclusion](file-inclusion.md)**: The include directive pulls files and server-side resources into an SSI-parsed page; file and virtual paths plus traversal let an attacker read local files a...

## Tools

- **Burp Suite**: reflecting and iterating injected SSI directives through Repeater and Intruder.
- **Burp Collaborator**: confirming blind directive execution out-of-band.
- Manual testing with crafted `<!--#...-->` directives.

## References

- OWASP: Server-Side Includes (SSI) Injection
- OWASP WSTG: Testing for Server-Side Includes (SSI) Injection
