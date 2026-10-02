---
title: "Server-Sent Events and long polling: auth gaps, session scope, and stream injection"
description: Exploiting SSE and long-poll transports—connect-time-only authorization, cross-user response sharing through proxy and CDN caching, topic/tenant parameter tampering, and injection carried through event-stream payloads.
keywords:
  - Server-Sent Events
  - SSE
  - long polling
  - EventSource
  - cache poisoning
  - stream injection
---

# SSE and long polling

Server-Sent Events (SSE) and long polling are the HTTP-native realtime transports. Both ride ordinary requests, so they inherit the web's caching, proxying, and session machinery—and they inherit its failure modes. SSE opens a single `GET` whose response stays open as `text/event-stream`, pushing `data:` events until the connection drops. Long polling issues a `GET` that the server holds until data is ready, then the client immediately reissues it. Because the authorization decision tends to be made **once**, when the stream opens, and because the responses look cacheable to intermediaries, these endpoints expose a distinct set of offensive opportunities.

> **Scope.** For authorized penetration tests, red-team engagements, CTF labs, and code review of systems you own or are contracted to assess. Testing streams without written authorization is unlawful.

## Overview

A representative SSE endpoint subscribes a client to a topic drawn straight from the query string:

```js
app.get("/events", (req, res) => {
  res.set("Content-Type", "text/event-stream");
  const topic = req.query.topic;               // e.g. "user.42.feed"
  bus.subscribe(topic, (evt) => res.write(`data: ${JSON.stringify(evt)}\n\n`));
});
```

The user is known from the session cookie, but `topic` is never checked against that user. Whoever can reach the endpoint streams whatever topic they name. The long-poll variant has the same shape—`GET /poll?channel=...&since=...`—and the same question applies to every open: *is tenant and topic ownership re-resolved on this request, or assumed from a prior one?*

## Attack surface

### Connect-time-only authorization and parameter tampering

The `EventSource` URL and the long-poll URL carry the routing parameters—`topic`, `channel`, `room`, `tenant`, `since`—in the query string. Because authorization is usually evaluated at open and not re-bound to the authenticated principal, tampering with those parameters is the primary primitive:

```
# authenticated as a low-privilege user
GET /events?topic=user.<VICTIM_ID>.feed
GET /poll?channel=tenant/<OTHER_TENANT>/alerts&since=0
```

If the victim's events arrive on your stream, topic authorization is missing. This is the SSE/long-poll form of BOLA: one open yields a continuous feed rather than a single record. The `since`/cursor parameter is worth tampering with too—setting it to `0` or an old value often replays historical events the current session would never otherwise see.

### Cross-user caching and buffering

SSE and long-poll responses pass through forward proxies, reverse proxies, and CDNs that may **buffer** or **cache** them incorrectly:

- **Shared cache serving another user's stream.** If a per-user `/events` or `/poll` response is cached without `Cache-Control: no-store` and without a `Vary` that accounts for the session, an intermediary can hand user A's buffered events to user B. Probe by requesting the endpoint from two sessions and comparing bodies, and by inspecting `Age`, `X-Cache`, and `CF-Cache-Status` response headers for a hit.
- **Cache key confusion / web cache deception.** Appending a static-looking suffix (`/events/x.css`, `/poll;foo.js`) can trick a CDN into caching a dynamic per-user stream under a key the attacker can later fetch, exposing the victim's buffered events.
- **Proxy buffering and boundary leakage.** Intermediaries that buffer `text/event-stream` can merge or split events across the `\n\n` boundary, occasionally stitching one user's trailing event onto another connection when pooling is misconfigured.

### Session scope and long-poll deduplication

Long-poll backends frequently **deduplicate** identical in-flight requests or **share** a pending response across clients to save work. If the dedup key is the channel alone and ignores the session, two users polling the same channel name can be served the same held response—so a guessed or shared channel id leaks events across accounts. Confirm by having two lab sessions poll an identical channel and watching whether a single server event is delivered to the wrong principal.

## Exploitation

### Step 1 — map the transport and its parameters

Open the client with a proxy attached. For SSE, capture the `EventSource` URL and note every routing parameter; for long poll, capture the `GET` loop and its cursor parameter. Record the response headers that govern caching (`Cache-Control`, `Vary`, `Age`, CDN cache markers).

### Step 2 — tamper for cross-tenant reads

Swap `topic`/`channel`/`tenant` values to another lab user's identifiers and rewind the cursor:

```
GET /events?topic=tenant/<OTHER>/orders
GET /poll?channel=user.<VICTIM>.dm&since=0
```

A stream of foreign events confirms connect-time-only authorization.

### Step 3 — probe the caching layer

Request the per-user stream twice from different sessions and diff the bodies; a match plus a cache-hit header indicates cross-user cache exposure. Try web-cache-deception suffixes to force caching under an attacker-fetchable key.

### Injection through the event stream

SSE and long-poll payloads are just another path into the application's **service layer**: the JSON the client sends to establish filters, and the data the server reflects into `data:` events, feed the same sinks as REST handlers. Two directions matter:

- **Inbound filter/parameter → sink.** Where the subscription accepts a filter expression, search term, or `since` token that the handler concatenates into a query or command, the usual SQL, command, and path primitives apply. See [Message injection to downstream sinks](message-injection-to-downstream-sinks.md).
- **Outbound event → client-side sink.** Attacker-influenced content streamed back as `data:` is parsed and often rendered by the client. If the client injects event payloads into the DOM without encoding, a stored value echoed through the stream becomes DOM-based XSS—delivered over a channel defenders rarely inspect. The `event:` and `id:` fields are equally attacker-reachable when they derive from stored data.

## Practical notes

- **SSE is same-origin by default, but CORS can open it.** `EventSource` is subject to CORS; an endpoint that reflects `Access-Control-Allow-Origin` with credentials widens who can open the stream cross-site.
- **Reconnect replays.** `EventSource` automatically reconnects and sends `Last-Event-ID`. A server that trusts that header to resume a stream can be fed a crafted id to request arbitrary historical positions.
- **Compression and buffering flush.** Some stacks only flush events after a buffer fills; forcing many small events can reveal whether an intermediary is coalescing streams across connections.

## Tools

- **[Burp Suite](https://portswigger.net/burp)** — intercept the `EventSource`/long-poll requests, replay with tampered parameters, and inspect cache headers.
- **[curl](https://curl.se/)** — `curl -N https://target/events?topic=...` streams SSE raw for scripted parameter fuzzing and header inspection.
- **[Param Miner](https://github.com/PortSwigger/param-miner)** — discover unkeyed inputs and cache-key quirks relevant to cross-user cache exposure.
- A second authenticated session to diff per-user responses and confirm shared-cache or dedup leakage.

## References

- [HTML Living Standard: Server-Sent Events](https://html.spec.whatwg.org/multipage/server-sent-events.html)
- [PortSwigger Web Security Academy: Web cache deception](https://portswigger.net/web-security/web-cache-deception)
- [OWASP API Security Top 10: API1 Broken Object Level Authorization](https://owasp.org/API-Security/editions/2023/en/0xa1-broken-object-level-authorization/)
- [CWE-525: Use of Web Browser Cache Containing Sensitive Information](https://cwe.mitre.org/data/definitions/525.html)
