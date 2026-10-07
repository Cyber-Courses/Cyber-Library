---
title: "Bitbucket"
order: 3
description: "Offensive techniques against Bitbucket Cloud as a SaaS target: workspace, project, and repository reconnaissance, secret harvesting from code and history, Pipelines execution with repository and secured variables, deployment and OIDC abuse, and app-password and access-token reuse against the 2.0 API."
keywords:
  - Bitbucket
  - Bitbucket Pipelines
  - app password
  - access token
  - bitbucket-pipelines.yml
---

# Bitbucket

Bitbucket Cloud is a code forge reached through its **REST API** (base `https://api.bitbucket.org/2.0`) and the `git` protocol over HTTPS. Its hierarchy is **workspace -> project -> repository**: a workspace is the top-level account that owns repositories and billing, projects are an optional grouping layer inside it, and permissions are granted at any of the three. Almost every attack is a question of **what a credential can read across that hierarchy** and of **what runs when code enters a repository**, because **Pipelines** executes `bitbucket-pipelines.yml` with the repository's variables and deployment credentials.

Credentials come in a few distinct shapes, and which one you hold changes everything downstream: **app passwords** (Basic auth, `username:app_password`, scoped per user), **repository, project, and workspace access tokens** (Bearer auth, scoped to one resource), **Atlassian API tokens** (Basic auth with the account email), and **OAuth consumers**. The recurring chain is: enumerate the workspace and its repositories, harvest secrets from clones and history, get code execution in a pipeline, then reuse the tokens that execution exposes to push, deploy, and pivot into connected cloud accounts.

These pages target `bitbucket.org` (Cloud). Bitbucket Data Center/Server is a different product with a different API (`/rest/api/1.0/`) and no Pipelines; differences are called out inline where a technique changes.

## Triage

Establish the identity you hold and the workspace surface before anything else:

```bash
# What identity does this credential map to
curl -s -u "$BB_USER:$BB_APP_PASSWORD" https://api.bitbucket.org/2.0/user | jq '{username,account_id,nickname}'

# Workspaces this credential can see, then repositories in each (public + private it can reach)
curl -s -u "$BB_USER:$BB_APP_PASSWORD" https://api.bitbucket.org/2.0/workspaces | jq -r '.values[].slug'
curl -s -u "$BB_USER:$BB_APP_PASSWORD" \
  "https://api.bitbucket.org/2.0/repositories/ACME?pagelen=100" | jq -r '.values[].full_name'
```

A credential that returns private repositories, or whose `/2.0/user` call succeeds at all, is already a foothold. Repositories that contain a `bitbucket-pipelines.yml` are the CI attack surface; those with configured **deployment environments** or **repository variables** are the high-value ones because a pipeline step reads them in cleartext.

Pick the page by the access you have: public exposure, a clone you can read, or a `git` URL routes to **Reconnaissance**; a repository whose pipeline you can trigger routes to **Pipelines abuse**; a leaked app password or access token routes to **Token and app-password abuse**.

## Pages

- **[Reconnaissance](reconnaissance.md)**: enumerating workspaces, projects, repositories, and members through the 2.0 API, reading source and the pipeline config, and mining secrets from full clone history manually and with history scanners.
- **[Pipelines abuse](pipelines-abuse.md)**: triggering `bitbucket-pipelines.yml` from a branch or pull request you can push, exfiltrating repository and secured variables by defeating log masking, and reaching deployment credentials and OIDC-brokered cloud roles.
- **[Token and app-password abuse](token-and-app-password-abuse.md)**: identifying and reusing leaked app passwords, repository/project/workspace access tokens, and Atlassian API tokens against the API and `git`, reading their identity and scope, and acting within it.

## References

- [Bitbucket Cloud REST API reference](https://developer.atlassian.com/cloud/bitbucket/rest/intro/)
- [Bitbucket Pipelines documentation](https://support.atlassian.com/bitbucket-cloud/docs/get-started-with-bitbucket-pipelines/)
- [HackTricks Cloud: Bitbucket Pipelines security](https://cloud.hacktricks.wiki/en/pentesting-ci-cd/bitbucket-security/index.html)
- [Rhino Security Labs: CI/CD pipeline attacks](https://rhinosecuritylabs.com/aws/circleci-attack-aws/)
