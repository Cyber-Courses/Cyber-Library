---
title: "GitLab"
order: 2
description: "Attacking GitLab as a hosted forge: enumerating groups, projects, members, and CI/CD variables through the v4 REST and GraphQL APIs, injecting jobs into .gitlab-ci.yml, taking over shared and Docker-executor runners, abusing personal, project, group, deploy, and CI job tokens, and leveraging published self-managed account-takeover and upload RCE weaknesses."
keywords:
  - GitLab
  - gitlab-ci.yml
  - CI job token
  - personal access token
  - runner
  - api/v4
---

# GitLab

GitLab stores an organization's source, its automation, and the credentials that automation runs with, so it is a direct route into the software supply chain. On gitlab.com everything is reached through the web UI, the REST API at `https://gitlab.com/api/v4`, and GraphQL at `https://gitlab.com/api/graphql`; self-managed instances expose the same APIs under their own host. Work is organized into **groups** (which nest and own membership and CI/CD variables inherited by their projects) and **projects** (repositories with pipelines, registries, and their own variables). The lever that turns read access into execution is **CI/CD**: a pipeline defined by `.gitlab-ci.yml` runs your `script:` on a **runner** with the project's and group's CI/CD variables in the environment and a `CI_JOB_TOKEN` that reaches the API and registry.

Authentication to the API is a header, one of several token types: a **personal access token** (`PRIVATE-TOKEN:`), a **project/group access token**, a **deploy token**, an **OAuth** bearer, or the pipeline-scoped **CI job token** (`JOB-TOKEN:`). Any one of them, leaked or over-scoped, is the starting point.

## Triage

```bash
# What a token (or anonymous access) can see
curl -s https://gitlab.com/api/v4/version -H "PRIVATE-TOKEN: $TOKEN"        # token valid + instance version
curl -s "https://gitlab.com/api/v4/groups?per_page=100" -H "PRIVATE-TOKEN: $TOKEN"
curl -s "https://gitlab.com/api/v4/projects?membership=true&per_page=100" -H "PRIVATE-TOKEN: $TOKEN"
# Anonymous public inventory (no token): public projects/groups still enumerate
curl -s "https://gitlab.com/api/v4/groups?search=<org>"
```

A `200` with group/project JSON means the token (or anonymous access) sees that scope; a `401` means the token is invalid, a `403` means it lacks the scope or role. Pick the page by what you hold: public exposure and leaked secrets go to **Reconnaissance**; a branch or MR whose pipeline you can run goes to **CI/CD pipeline injection**; a runner you can reach through a job goes to **Runner takeover**; a token in hand goes to **Token abuse**; a self-managed instance at a known-vulnerable version goes to **Known admin and API exploits**.

## Pages

- **[Reconnaissance](reconnaissance.md)**: enumerate groups, projects, members, snippets, and accessible CI/CD variables through v4 and GraphQL, and harvest secrets from repositories, history, and job logs.
- **[CI/CD pipeline injection](ci-cd-pipeline-injection.md)**: run attacker `script:` in a pipeline to exfiltrate protected and masked variables and pivot with the `CI_JOB_TOKEN`.
- **[Token abuse](token-abuse.md)**: reuse leaked personal, project, group, deploy, and CI job tokens against the API, resolving each token's identity and scope.
- **[Known admin and API exploits](known-admin-and-api-exploits.md)**: published self-managed weaknesses by mechanism, including password-reset account takeover, image-upload RCE, and API/GraphQL IDOR and SSRF classes.
- **[Runner takeover](runner-takeover.md)**: get RCE on shared runners through a controlled job, break out of the Docker executor, and register a rogue runner with a leaked token.

## References

- [GitLab REST API](https://docs.gitlab.com/ee/api/rest/)
- [GitLab CI/CD variables](https://docs.gitlab.com/ee/ci/variables/)
- [GitLab GraphQL API](https://docs.gitlab.com/ee/api/graphql/)
- [PortSwigger Research](https://portswigger.net/research)
