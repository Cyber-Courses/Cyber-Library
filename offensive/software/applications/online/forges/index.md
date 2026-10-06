---
title: "Forges: attacking SaaS code-hosting platforms"
description: "Attacking hosted code forges (GitHub, GitLab, Bitbucket) as vendor-operated services: reconnaissance and secret harvesting across organizations and repositories, CI/CD pipeline injection, self-hosted runner takeover, and token, OAuth app, and webhook abuse that turn repository access into code execution and supply-chain compromise."
keywords:
  - code forge
  - GitHub
  - GitLab
  - Bitbucket
  - CI/CD
  - supply chain
---

# Forges

A code forge hosts an organization's source, its automation, and the credentials that automation runs with, so compromising one is a direct route into the software supply chain. These are vendor-operated SaaS platforms: there is no server to exploit, you work through the web UI, the REST and GraphQL APIs, and above all the CI/CD system, which runs attacker-influenced code with repository secrets and deploy credentials. The recurring chain is the same across all three: find the org and its repositories, harvest secrets from code and history, get code execution in a pipeline, and turn a leaked or over-scoped token into broader access.

## Triage

```bash
# Identify the forge and whether org/repo data is reachable unauthenticated
curl -s https://api.github.com/orgs/<org>            # GitHub: public org + repo metadata
curl -s https://gitlab.com/api/v4/projects?search=<org>   # GitLab: project search
curl -s https://api.bitbucket.org/2.0/repositories/<workspace>  # Bitbucket: workspace repos
# then decide the lever: exposed secrets (recon), a pipeline you can influence (CI/CD), or a token you hold
```

Pick the subtopic by the access you have: public exposure and leaked secrets route to each forge's **Reconnaissance**; a repository whose workflow you can influence routes to the **CI/CD injection** and **runner** pages; a token or app credential routes to the **token abuse** pages.

## Subtopics

- **[GitHub](github/index.md)**: reconnaissance and secret harvesting, Actions workflow injection, self-hosted runner takeover, `GITHUB_TOKEN` and PAT abuse, and OAuth App and webhook abuse.
- **[GitLab](gitlab/index.md)**: reconnaissance, `.gitlab-ci.yml` pipeline injection, runner takeover, personal-access and CI job token abuse, and known admin and API exploits.
- **[Bitbucket](bitbucket/index.md)**: reconnaissance, Pipelines abuse, and app-password and access-token abuse.

## References

- [GitHub Actions security hardening](https://docs.github.com/en/actions/security-guides/security-hardening-for-github-actions)
- [GitLab CI/CD documentation](https://docs.gitlab.com/ee/ci/)
- [PortSwigger Research: CI/CD and GitHub Actions attacks](https://portswigger.net/research)
