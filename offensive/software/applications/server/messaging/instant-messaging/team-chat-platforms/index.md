---
title: "Team-chat platforms: attacking self-hosted chat servers through their REST API and integrations"
description: "Self-hosted team chat (Mattermost, Rocket.Chat) is a web app plus a REST API and a websocket over a Postgres or Mongo store. The attack surface is API authentication, broken authorization (IDOR across teams and channels), injection into the auth and query layer, and the admin-only integration, webhook, and plugin features that reach code execution on the server. Covers fingerprinting the product and version and where to go from there."
keywords:
  - team chat
  - mattermost
  - rocket.chat
  - chat server api
  - integration rce
---

# Team-chat platforms

A self-hosted team-chat product is three services behind one hostname: a web application that renders the client, a **REST API** that does all the real work, and a **websocket** that streams live events, all backed by a relational or document database (Postgres or MySQL for Mattermost, MongoDB for Rocket.Chat). Nearly every offensive primitive lives in the REST API rather than the browser UI, because the UI is just one client of that API and the API is reachable directly with `curl`. That shape sets the four recurring questions: can you authenticate or abuse the auth layer, can you reach other teams' and channels' data through broken authorization (IDOR), can you inject into the query or login layer, and can you reach the admin-only integration, webhook, or plugin machinery that runs attacker code on the server.

The decisive property is that both products ship powerful server-side extension features to administrators: Mattermost loads uploaded Go plugins into the server process, and Rocket.Chat runs integration "scripts" in a sandbox that has historically been escapable to `require`/`process`. So the end state of a chat-server compromise is rarely "read some messages"; it is code execution as the service account once you hold an admin token, and the auth layer is where that token is won.

This area is self-hosted products only (the web app plus its REST API on your target), not the SaaS Slack, Teams, or Discord clouds.

## Triage: fingerprint the product and version

One unauthenticated sweep separates the two products and pins the build, which decides which auth bypass and which integration path is live:

```bash
# Mattermost: Go server, default 8065, REST under /api/v4
curl -sk http://<target>:8065/api/v4/system/ping
#   {"status":"OK"} (and an X-Version-Id response header on most routes)
curl -skI http://<target>:8065/api/v4/users | grep -i x-version-id
#   X-Version-Id: 9.5.0.9.5.0.<hash>.true   -> exact build
curl -sk 'http://<target>:8065/api/v4/config/client?format=old' | tr ',' '\n' | grep -i version
#   "Version":"9.5.0","BuildNumber":...     unauthenticated client config

# Rocket.Chat: Node/Meteor, default 3000, REST under /api/v1
curl -sk http://<target>:3000/api/info
#   {"version":"6.5.0","success":true}      unauthenticated version
curl -skI http://<target>:3000/sockjs/info  # Meteor/SockJS transport confirms Rocket.Chat/Meteor
```

Read the signals: a `/api/v4/system/ping` returning `{"status":"OK"}` and an `X-Version-Id` header is Mattermost; a `/api/info` JSON `version` plus a `/sockjs/` DDP transport and the `X-Instance-Id` header is Rocket.Chat. The static asset path (`/static/` for Mattermost's Go-served bundle) and the login markup confirm it. Record the exact version before touching the auth layer, because the NoSQL login bypass and the plugin/integration chains are version-bound.

## Subtopics

- **[Mattermost](mattermost/index.md)**: Go server with the `/api/v4` REST API over Postgres/MySQL. Enumeration through `/api/v4/users` and `/api/v4/teams`, the login and personal-access-token auth layer, and server exploitation through API IDOR, filter-parameter SQL injection, and signed/unsigned Go plugin upload to code execution.
- **[Rocket.Chat](rocket-chat/index.md)**: Node/Meteor server with the `/api/v1` REST API and DDP over MongoDB. Enumeration through `users.list`/`spotlight`, the NoSQL operator-injection login bypass, and server exploitation by chaining that bypass to an admin token and the integration-script sandbox escape for RCE.

## References

- [Mattermost API reference (`/api/v4`)](https://api.mattermost.com/)
- [Rocket.Chat REST API reference (`/api/v1`)](https://developer.rocket.chat/apidocs)
- [HackTricks: pentesting network services](https://book.hacktricks.wiki/en/network-services-pentesting/index.html)
- [OWASP Testing Guide: testing for NoSQL injection](https://owasp.org/www-project-web-security-testing-guide/)
