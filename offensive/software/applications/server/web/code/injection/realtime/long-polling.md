---
title: "Long polling"
order: 3
description: "Held-open and poll endpoints where JSON notifications, poll query params, and session tokens in the poll URL reach server-side queries and authorization."
keywords:
  - long polling
  - comet
  - poll endpoint
  - token in URL
  - injection
---

# Long polling

Long polling emulates a push channel over ordinary HTTP: the client issues a request that the server holds open until data is available or a timeout fires, then the client immediately re-requests. Each cycle is a normal HTTP request, which means long-polling endpoints inherit the full REST attack surface, plus two problems specific to the pattern: the values that drive the poll (cursor, channel, token) travel on every cycle and often sit in the URL, and the held-open, re-request loop exposes ordering and timing behavior an attacker can steer.

## Injection through poll parameters

A poll request carries state telling the server what the client wants next, typically a channel name, a cursor, and a since-timestamp:

```http
GET /poll?channel=user:42&since=1696000000&cursor=a1b2 HTTP/1.1
Host: target.example
Cookie: session=...
```

Every one of these parameters is a candidate sink. If `channel` is concatenated into a query that selects pending messages, it carries SQL:

```
/poll?channel=user:42' UNION SELECT secret FROM tokens-- -
```

If `since` or `cursor` feeds a document store lookup, an operator object submitted as JSON in a POST-based poll bypasses the filter:

```json
{"channel": "user:42", "cursor": {"$gt": ""}}
```

Because long-poll endpoints are hit continuously and are often treated as low-value plumbing, their parameters are frequently less scrutinized than primary REST routes while reaching the same backends.

## Channel as an access-control sink

The `channel` parameter is also an authorization boundary. If the server returns whatever channel the poll names without checking that the session may read it, swapping the channel reads other users' queued notifications over the attacker's own authenticated poll:

```
/poll?channel=user:43
/poll?channel=admin:events
/poll?channel=tenant:acme:billing
```

This is the poll-endpoint form of broken object-level authorization: identity is checked, the named channel is not.

## Token-in-URL leakage

Long-poll clients often carry the session or a per-channel token in the poll URL itself, because the pattern predates and sidesteps cleaner transport auth:

```
/poll?channel=user:42&access_token=eyJhbGciOi...
```

A token in the query string leaks through every layer that records request URLs: proxy and server access logs, and APM or analytics that capture full URLs. These are real harvesting points, and a shared or ingested access log hands the token to anyone who can read the logs. The browser-side leaks (history and `Referer` to third-party resources) apply only when the token sits in the **top-level page URL**: for an XHR or `fetch` poll the request URL is never navigated to, so it is not added to history, and a later third-party resource receives the containing page's URL as `Referer`, not the poll URL.

## Ordering and timing

The held-open, re-request loop creates a race surface. Because the client fires a fresh poll the instant the previous one returns, two polls can overlap during the re-request gap, and an attacker who controls both the poll and a concurrent state-changing request can interleave them to read a value mid-update or to receive a notification intended to be gated behind a check that has not yet committed. Delivery ordering is also observable: by measuring which poll returns a given record first, an attacker can infer server-side sequencing and the timing of events in other users' sessions that share a backend queue.

## Finding the sinks

Capture one poll cycle, then enumerate each parameter against the same payloads used for REST routes, and separately swap the channel and cursor to neighboring and privileged values. Check whether the session or a token rides in the URL rather than a cookie or header; if so it leaks to server and proxy logs, and additionally through browser history or `Referer` only when that same token also appears in the top-level page URL.

## Tools

- **Burp Suite**: intercept a poll cycle, then fuzz and replay parameters with Burp Repeater.
- **curl**: issue poll requests directly to swap channel, cursor, and token values.

## References

- PortSwigger Web Security Academy: Access control vulnerabilities and privilege escalation
- OWASP Web Security Testing Guide: Testing for Insecure Direct Object References
