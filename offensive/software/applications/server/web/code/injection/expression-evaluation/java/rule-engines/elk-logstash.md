---
title: "ELK Logstash injection: code execution through the ruby filter"
description: "The Logstash pipeline ruby filter executes arbitrary Ruby supplied in its code option, so configuration or expression injection into a ruby filter is Ruby execution and command execution on the pipeline host."
keywords:
  - Logstash
  - Logstash ruby filter
  - ELK injection
  - pipeline injection
  - Ruby code execution
---

# ELK Logstash

Logstash is the ingestion and transformation stage of the ELK stack. A pipeline is a configuration of input, filter, and output plugins, and the filter stage can run arbitrary logic. The `ruby` filter is the sharp edge: its `code` option is a string of Ruby that Logstash compiles and runs for every event. Anything that lets untrusted input reach a pipeline definition, or reach the `code` of a `ruby` filter, is Ruby execution on the host running Logstash, which runs commands directly.

## The ruby filter as the execution path

A `ruby` filter embeds Ruby that executes per event:

```ruby
filter {
  ruby {
    code => "event.set('x', system('id'))"
  }
}
```

Where a pipeline is assembled from templates, stored in a datastore an attacker can write, or built from user-controlled fields (a multi-tenant pipeline builder, a UI that lets operators paste filter snippets), injecting into the `code` string runs attacker Ruby. Ruby reaches the operating system through several equivalents, so the command sink is flexible:

```ruby
system('id')          # runs and returns status
`id`                  # backticks capture stdout
IO.popen('id').read   # capture with a handle
require 'open3'; Open3.capture2('id')
```

Unlike the JVM engines, Ruby's backticks and `system('shell string')` do invoke a shell, so `$(...)`, pipes, and redirection expand normally here:

```ruby
`id > /tmp/o 2>&1; cat /etc/passwd | head`
```

The `ruby` filter can also load a standalone script through its `path` option, so where the attacker can place or point at a script file, that is an equally direct route. Capturing output back into `event.set(...)` surfaces command results into the indexed documents, which can then be read downstream.

## Conditionals are a weaker surface

Logstash pipeline conditionals (`if [field] == "..."`) support a comparison and boolean grammar over event fields but are not a scripting language. Injection into a conditional lets an attacker alter routing and filter logic, drop or misroute events, or branch on crafted field values, but it does not by itself execute code. Treat conditionals as a logic-tampering surface and the `ruby` filter as the code-execution path.

## Tools

- Manual testing with Burp Repeater; payloads crafted per engine.
- **Burp Suite**: Repeater to inject into pipeline or ruby-filter fields exposed over HTTP.

## References

- [Logstash: ruby filter plugin](https://www.elastic.co/guide/en/logstash/current/plugins-filters-ruby.html)
- [Logstash: pipeline configuration](https://www.elastic.co/guide/en/logstash/current/configuration.html)
- [Logstash: conditionals](https://www.elastic.co/guide/en/logstash/current/event-dependent-configuration.html#conditionals)
