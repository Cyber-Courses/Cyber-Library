---
title: "Mattermost: attacking the /api/v4 REST server and its plugin layer"
description: "Attacking a self-hosted Mattermost server: a Go process on 8065 exposing the /api/v4 REST API over Postgres or MySQL. Fingerprint the build from /api/v4/system/ping and the X-Version-Id header, then move through user and team enumeration, the login and personal-access-token auth layer, and server exploitation via API IDOR, filter-parameter SQL injection, and Go plugin upload to code execution."
keywords:
  - mattermost
  - mattermost api
  - mattermost plugin rce
  - api v4
  - x-version-id
---

# Mattermost

Mattermost is a Go server that serves a React client and does all of its work through a single REST API rooted at `/api/v4`, listening on `8065` by default (often reverse-proxied to 443), backed by PostgreSQL or MySQL. Because the Go binary is one process that both serves the API and loads server-side plugins, the high-value path is: get a token through the auth layer, find an account (or a bug) that reaches system-admin, then upload a Go plugin that runs as the `mattermost` service account. Everything the client does maps to a documented `/api/v4` call you can replay with `curl` and an `Authorization: Bearer` header.

## Fingerprint first

Confirm the product and pin the exact build, because the SQL-injection filter params and the plugin-signature behavior are version-bound.

```bash
curl -sk http://<target>:8065/api/v4/system/ping
#   {"status":"OK"}                      confirms Mattermost
curl -skI http://<target>:8065/api/v4/users | grep -i x-version-id
#   X-Version-Id: 9.5.0.9.5.0.<hash>.true    major.minor.patch + build hash + "enterprise" flag
curl -sk 'http://<target>:8065/api/v4/config/client?format=old' | python3 -m json.tool | grep -iE 'version|buildnumber|enablesignup|enableopenserver'
```

The unauthenticated `config/client?format=old` is the single richest fingerprint: it returns the full client configuration, including `Version`, whether open signup and open-server registration are on (`EnableOpenServer`, `EnableUserCreation`), the SSO providers wired up, and the configured `SiteURL`. The trailing `.true`/`.false` in `X-Version-Id` tells you whether this is the Enterprise build, which changes which endpoints exist. Record all of it before authenticating.

## Pages

- **[Enumeration](enumeration.md)**: harvesting users, teams, and channels through `/api/v4/users`, `/api/v4/users/search`, and `/api/v4/teams`, confirming account existence via `/api/v4/users/email/{email}` and login-error differences, and reading `X-Version-Id` to scope later exploitation.
- **[Authentication](authentication.md)**: the `POST /api/v4/users/login` flow that returns a session in the `Token` header and the `MMAUTHTOKEN` cookie, password spraying, personal access tokens, SSO and MFA gaps, and the password-reset endpoint.
- **[Server exploitation](server-exploitation.md)**: API authorization flaws and IDOR across teams, SQL injection in API filter parameters, path traversal in the file and import endpoints, and the system-admin plugin upload (`/api/v4/plugins`) that executes a Go plugin on the server.

## References

- [Mattermost API reference (`/api/v4`)](https://api.mattermost.com/)
- [Mattermost plugin developer documentation](https://developers.mattermost.com/integrate/plugins/)
- [mattermost/mattermost (server source)](https://github.com/mattermost/mattermost)
- [Mattermost security updates](https://mattermost.com/security-updates/)
