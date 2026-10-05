---
title: "Enumeration: mapping a homeserver over the Matrix HTTP APIs"
description: "The Matrix client-server and federation APIs answer unauthenticated probes with curl: .well-known files reveal the real backend, the register endpoint discloses whether registration is open, the Synapse admin path reveals its presence, and the user-directory and public-rooms endpoints list accounts and rooms. The federation profile query returns user existence and display names from another server's perspective."
keywords:
  - Matrix enumeration
  - well-known delegation
  - user directory
  - publicRooms
  - federation query
---

# Enumeration

Everything in Matrix is an HTTP request, so a homeserver is enumerated entirely with `curl`. Three unauthenticated surfaces matter: delegation (`.well-known`) tells you the true backend host and port behind any proxy; the registration and admin endpoints tell you how the server can be entered; and the directory and federation query endpoints list users and rooms. The directory search and public-rooms listing often answer without a token, and the federation profile query lets you confirm user existence and read display names from outside the server entirely.

## Preconditions

Network reach to 443/8008 (client API) and ideally 8448 (federation). Read `/_matrix/client/versions` first to confirm it is a Matrix homeserver and to see which spec versions and unstable features are enabled. Some endpoints (directory search in particular) may require a token on hardened deployments; if they return `M_MISSING_TOKEN` or `M_UNKNOWN_TOKEN`, provision an account via [authentication](authentication.md) and retry with `Authorization: Bearer <token>`.

## Delegation and entry points

```bash
# where the real server lives (proxy fronting)
curl -s https://target.lan/.well-known/matrix/server
# {"m.server":"matrix-internal.target.lan:8448"}  => the true federation backend

# is registration open? ask the register endpoint what flows it offers
curl -s -X POST https://target.lan/_matrix/client/v3/register \
  -H 'Content-Type: application/json' -d '{}'
# {"flows":[{"stages":["m.login.dummy"]}], "session":"..."}  => OPEN registration (dummy stage only)
# {"errcode":"M_FORBIDDEN","error":"Registration has been disabled"}  => closed

# is this Synapse with its admin API exposed?
curl -s -o /dev/null -w "%{http_code}\n" https://target.lan/_synapse/admin/v1/server_version
# 200 => Synapse admin API reachable; 404 => not Synapse or admin path not routed
```

Interpretation: a `flows` response containing only `m.login.dummy` means anyone can register with no email or token (go straight to [authentication](authentication.md)); a `session` with a `m.login.recaptcha`/`m.login.email.identity` stage means registration is open but gated. A `200` on `/_synapse/admin/v1/server_version` confirms Synapse and that the admin surface is routed, which is the target for shared-secret abuse.

## User and room directories

```bash
# search the user directory (often token-gated; use a Bearer token if required)
curl -s -X POST https://target.lan/_matrix/client/v3/user_directory/search \
  -H 'Authorization: Bearer <token>' -H 'Content-Type: application/json' \
  -d '{"search_term":"a","limit":100}'
# returns {"results":[{"user_id":"@jdoe:target.lan","display_name":"John Doe"}], ...}

# list published rooms (frequently answers unauthenticated)
curl -s https://target.lan/_matrix/client/v3/publicRooms
# each chunk entry has room_id, name, topic, num_joined_members, and whether it is world-readable
```

Searching the directory for common letters enumerates accounts and display names, building the user inventory; the public-rooms chunk names rooms, their topics (which leak internal detail), and member counts, and flags world-readable rooms whose history you can read on joining.

## Federation profile query

```bash
# confirm a user exists and read their profile from another server's point of view
curl -s "https://target.lan:8448/_matrix/federation/v1/query/profile?user_id=@jdoe:target.lan&field=displayname"
# {"displayname":"John Doe"}  => the account exists
# {"errcode":"M_NOT_FOUND"}   => no such user
```

The federation profile query needs no local account and confirms whether a given `@user:domain` exists on the homeserver, so a name list over this endpoint enumerates valid accounts even when the client-side directory is locked down.

## Follow-on

- An open `register` flow is an immediate credential; proceed to [authentication](authentication.md) to mint an account and, on Synapse, an admin.
- Enumerated `@user:domain` identifiers feed impersonation, direct-message phishing, and the admin user-management API once you hold admin.
- World-readable public rooms give readable history without joining; scrape topics and pinned messages for secrets.

## References

- [Matrix client-server API (registration and directories)](https://spec.matrix.org/latest/client-server-api/)
- [Matrix server-server (federation) API](https://spec.matrix.org/latest/server-server-api/)
- [Synapse admin API documentation](https://element-hq.github.io/synapse/latest/usage/administration/admin_api/index.html)
