---
title: "Server-Sent Events"
order: 2
description: "Newline injection into a text/event-stream where server-built data, event, and id lines come from untrusted state, forging events and poisoning shared streams."
keywords:
  - Server-Sent Events
  - SSE injection
  - event-stream
  - newline injection
  - stream poisoning
---

# Server-Sent Events

Server-Sent Events push a one-way stream of text from server to client over a single long-lived response with content type `text/event-stream`. The wire format is deliberately simple: each event is a group of lines like `data:`, `event:`, and `id:`, and a blank line terminates one event and starts the next. That simplicity is the weakness. The boundaries between events are plain newlines in the body, so when the server builds those lines from untrusted state without stripping line breaks, attacker input can write its own stream structure.

## The stream format

A well-formed event looks like this:

```
event: message
data: {"user": "alice", "text": "hello"}
id: 42

```

The client's `EventSource` parser splits on newlines and dispatches on the blank-line boundary. The parser has no framing beyond these characters, so whoever controls a byte sequence that includes `\n` controls where events begin and end.

## Newline injection to forge events

Suppose the server echoes a username or message into a `data:` line:

```
data: {"from": "USERINPUT", "text": "..."}
```

If `USERINPUT` can contain a newline, a crafted value closes the current event and injects a second, fully attacker-chosen one. Supplying this as the username:

```
alice"}

event: adminNotice
data: {"text":"Account verified. Visit https://attacker.example to confirm."}
```

produces two events on the stream: the truncated original and a forged `adminNotice` that the client dispatches as if the server emitted it. Clients that register handlers per event type (`source.addEventListener('adminNotice', ...)`) will act on the injected type.

## Splitting and smuggling records

The same primitive splits one logical record into several, or merges intended boundaries. Injecting a blank line early terminates an event before the server intended, dropping fields the client expected and desynchronizing a parser that assumes fixed structure. Injecting an `id:` line sets the stream's last-event-id:

```
value

id: 99999
data: injected
```

Because the client echoes the last id back in the `Last-Event-ID` header when it reconnects, a forged `id:` can plant a value that the server trusts on resume, reaching whatever lookup consumes that header on reconnection.

## Poisoning a shared or relayed stream

The impact multiplies when the stream is shared. A notification or activity feed that fans one server-side `text/event-stream` out to many subscribers, or a proxy that relays an upstream stream to connected clients, will carry an injected event to every recipient. One attacker-supplied value containing newline-delimited event lines becomes a forged event delivered to the whole audience of that channel:

```
ok

event: priceUpdate
data: {"symbol":"ACME","price":0.01}
```

Every subscriber whose client acts on `priceUpdate` receives the forged record. Where the relay also stores events for replay, the forged lines persist and are re-served to clients that connect later.

## Finding the sink

Identify any field that the server reflects into the stream: usernames, chat bodies, status strings, resource names. Submit a value carrying `\n`, `\r\n`, a blank line, and a second `data:`/`event:`/`id:` block, then watch the raw stream (a plain `curl -N` against the endpoint shows the unparsed bytes). A stream where your injected lines appear as their own events, rather than escaped inside the original `data:` value, confirms the sink.

## Tools

- **curl -N**: stream the `text/event-stream` response to watch the raw, unparsed bytes.
- **Burp Suite**: intercept and tamper with the requests that feed reflected fields into the stream.
- Manual testing with Burp Repeater and crafted payloads.

## References

- OWASP Web Security Testing Guide: Testing for HTTP Splitting and Smuggling
- PortSwigger Web Security Academy: HTTP response header injection
