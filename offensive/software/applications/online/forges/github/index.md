---
title: "GitHub: attacking organizations, repositories, Actions, and tokens"
order: 1
description: "Offensive techniques against GitHub as a SaaS target: org and repo reconnaissance, secret harvesting from history, Actions workflow injection, GITHUB_TOKEN and PAT abuse, OAuth App and webhook persistence, and self-hosted runner takeover."
keywords:
  - GitHub
  - GitHub Actions
  - GITHUB_TOKEN
  - workflow injection
  - self-hosted runner
---

# GitHub

GitHub is a code forge reached entirely through its **REST and GraphQL APIs** (base `https://api.github.com`) and the `git` protocol. Almost every attack is a question of **what a token can see and do**, and of **what runs when code or an event enters a repository**. The chain is consistent: enumerate the org and its repos, harvest secrets from clones and history, turn a CI pipeline into code execution, then abuse the tokens that execution exposes to push, deploy, and pivot into connected cloud accounts.

These pages target `github.com` (SaaS). Where a self-hosted GitHub Enterprise Server (GHES) behaves differently, it is called out inline.

## Triage

Establish the identity you hold and the org/repo surface before anything else:

```bash
# What does this token see, and what scopes does it carry
curl -sI -H "Authorization: token $GH_TOKEN" https://api.github.com/user | grep -i '^x-oauth-scopes:'
gh auth status

# Org inventory: members, repos (public + private the token can see), teams
gh api /orgs/ACME/repos --paginate -q '.[].full_name'
gh api /orgs/ACME/members --paginate -q '.[].login'
gh api /user/repos --paginate -q '.[] | select(.permissions.admin==true) | .full_name'
```

A token that returns private repos, or an `X-OAuth-Scopes` line containing `repo`, `workflow`, or `admin:org`, is already a strong position. Repos with a `.github/workflows/` directory are the CI attack surface; repos using `pull_request_target`, `workflow_run`, or `runs-on: self-hosted` are the high-value ones.

## Pages

- **[Reconnaissance](reconnaissance.md)**: enumerating orgs, repos, members, and forks through REST and GraphQL, GitHub code search, gists, and harvesting secrets from full clone history and dangling objects, manually and with TruffleHog and Gitleaks.
- **[Actions workflow injection](actions-workflow-injection.md)**: abusing `pull_request_target` and `workflow_run` to run attacker PR code with a privileged `GITHUB_TOKEN` and secrets, and shell script injection from untrusted context expressions, with secret exfiltration, poisoned pipeline execution, and cache poisoning.
- **[Token and GITHUB_TOKEN abuse](token-and-github-token-abuse.md)**: default `GITHUB_TOKEN` permissions and `permissions:` scoping, the `workflow` scope, leaked classic and fine-grained PAT usage, reading scopes from response headers, and trading an OIDC token for a cloud role.
- **[OAuth App and webhook abuse](oauth-app-and-webhook-abuse.md)**: persistence through authorized OAuth Apps and installed GitHub Apps with installation access tokens, consent phishing, and repo/org webhooks that exfiltrate event payloads.
- **[Self-hosted runner takeover](self-hosted-runner-takeover.md)**: gaining RCE on non-ephemeral self-hosted runners through PR workflows, registering rogue runners with a leaked token, looting the runner work and credential directories, and pivoting to the internal network.

## References

- [GitHub REST API documentation](https://docs.github.com/en/rest)
- [Security hardening for GitHub Actions](https://docs.github.com/en/actions/security-for-github-actions/security-guides/security-hardening-for-github-actions)
- [PortSwigger Research: GitHub Actions attack diary](https://www.synacktiv.com/en/publications/github-actions-exploitation-introduction.html)
- [Praetorian gato: GitHub Actions attack tooling](https://github.com/praetorian-inc/gato)
- [PayloadsAllTheThings: CI/CD](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/CI%20CD/README.md)
