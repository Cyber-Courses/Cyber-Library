---
title: "Injection vulnerabilities"
description: "Untrusted data bound into something the server executes, parses, stores, renders, or requests, organized by the dangerous primitive under attack."
keywords:
  - injection
  - command injection
  - SQL injection
  - NoSQL injection
  - code injection
---

# Injection

Injection is where untrusted data is bound into something the server **executes, parses, stores, renders, or requests**, a database query, an operating-system command, markup, a template runtime, an HTTP message, or an outbound URL. When the data crosses from the *data* plane into the *code* plane, the attacker controls part of what the interpreter does.

This tree is organized strictly by the **primitive under attack**, not by how the input arrives (query string, header, or body, that axis lives under input delivery). Each child names the primitive: database queries, OS commands, markup parsers, template engines, HTTP handling, outbound requests, and more.

## Subtopics

- **[API](api/index.md)**: Server code that parses and dispatches structured API requests decides what runs, for whom, and against which backend.
- **[Command](command/index.md)**: Application code builds an operating-system command from untrusted input, letting an attacker change which program runs or with what arguments, usually remot...
- **[Database](database/index.md)**: Untrusted input altering the query a server sends to its data store, SQL, NoSQL, ORM escape hatches, and directory services.
- **[Expression evaluation](expression-evaluation/index.md)**: When an application evaluates an attacker-influenced expression string at runtime, the expression language becomes a code path.
- **[File](file/index.md)**: Abuse of how a web application takes in, resolves, includes, or exports files, turning file handling into code execution, disclosure, or downstream interpret...
- **[HTTP](http/index.md)**: Exploiting the HTTP message itself: header-value abuse, CRLF response splitting, request smuggling when front and back ends disagree, and HTTP parameter poll...
- **[Log](log/index.md)**: Untrusted input written into log lines or fields, letting an attacker forge events, shift fields, and confuse the parsers, aggregators, and SIEM pipelines th...
- **[Mail](mail/index.md)**: Application-layer email abuse where untrusted input reaches message headers, the body, or the SMTP hand-off and is interpreted as part of the message rather...
- **[Markup](markup/index.md)**: Injection into server-side markup and XML processing languages: Server-Side Includes, XML parsing, XPath/XQuery/XSLT queries, and XXE entity abuse.
- **[Realtime](realtime/index.md)**: Long-lived channels where message bodies and metadata drive routing, authorization, and sinks, so individual frames carry injection past the HTTP request bou...
- **[Request forgery](request-forgery/index.md)**: When untrusted input steers an outbound request, an attacker reaches internal services, cloud metadata, and alternative URL schemes.
- **[Template Engine](template-engine/index.md)**: Exploiting server-side template injection: detecting the engine, escalating from a template expression to the host language runtime, and the sandbox limits t...
