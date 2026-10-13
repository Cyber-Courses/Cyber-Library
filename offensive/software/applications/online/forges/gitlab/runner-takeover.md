---
title: "Runner takeover"
order: 5
description: "Getting code execution on GitLab runners through a controlled CI job, escaping the Docker executor when it runs privileged or mounts the host socket, reaching co-tenant project data on a shared runner, and registering a rogue runner with a leaked registration token to capture future jobs."
keywords:
  - GitLab runner
  - shared runner
  - Docker executor
  - privileged mode
  - runner registration token
  - rogue runner
---

# Runner takeover

A GitLab runner executes the `script:` of whatever job it picks up, so a job you control is code execution on the runner by design. On a **shared** runner that serves many projects, that execution sits next to other tenants' build environments, caches, and artifacts. The escalation targets are the host under a Docker-executor runner, the data of co-tenant projects, and the ability to register your own runner so it harvests future jobs.

## Preconditions

- A project whose pipeline you can run (see [CI/CD pipeline injection](ci-cd-pipeline-injection.md)) that is assigned a **shared** or group runner.
- For a host escape: the Docker executor configured with `privileged = true`, or the host Docker socket mounted into job containers. Fingerprint from inside a job before anything else:

```yaml
recon:
  script:
    - id; cat /proc/1/cgroup              # am I in a container, as which user
    - ls -la /var/run/docker.sock || true # is the host socket mounted into the job
    - grep CapEff /proc/self/status        # capability set -> privileged container
    - mount | grep -E 'docker|host'        # host paths bind-mounted in
```

A `docker.sock` present in the job, or a full `CapEff` capability mask, signals a privileged executor and a direct path to the host. A minimal capability set and no socket means you are confined to the container and should pivot to tenant data and rogue registration instead.

## Why the Docker executor escapes

The runner's `config.toml` sets the executor behavior. Two misconfigurations collapse the container boundary:

- `privileged = true` gives job containers the capabilities and device access to mount the host's block devices, so the "isolation" is cosmetic.
- A `volumes = ["/var/run/docker.sock:/var/run/docker.sock"]` mount hands the job control of the host's Docker daemon, which can start new containers that bind-mount the host filesystem.

Either one means a job can reach the host root filesystem. The concrete escape is the standard privileged-container / mounted-socket breakout documented in container-escape references; from the host you reach the runner's own `config.toml`, which contains the runner tokens, and any credentials cached on the box. Treat reading that `config.toml` as the objective, because it yields the runner authentication token used in the rogue-runner step below.

## Co-tenant data on a shared runner

Even confined to the container, a shared runner's working and cache trees persist between jobs and hold other projects' checkouts and cache archives:

```yaml
survey:
  script:
    - find / -type d \( -name builds -o -name cache \) 2>/dev/null
    - ls -la /builds 2>/dev/null; ls -la /cache 2>/dev/null
```

A populated `/builds/<other-group>` or `/cache` from projects that are not yours is co-tenant leakage: their source, and any secret a prior job wrote to disk, are readable from your job. Cache archives are also a poisoning vector, because a later job that restores a cache key you can write trusts its contents.

## Register a rogue runner

If you recover a **runner registration token** (from the host `config.toml`, a leaked group/project setting, or an admin foothold), you register your own runner against the instance. It then picks up real jobs from that scope and runs them in an environment you fully observe, which captures those jobs' CI/CD variables and job tokens as they execute.

```bash
# Register an attacker-controlled runner to the group/instance scope
gitlab-runner register \
  --non-interactive \
  --url "https://gitlab.com/" \
  --registration-token "$LEAKED_REG_TOKEN" \
  --executor shell \
  --description "build-cache-node-7"   # blend in with existing runner names

# Then run it; jobs it accepts execute on your host with their secrets in the env
gitlab-runner run
```

Interpreting it: a successful registration returns a runner authentication token and the runner appears in the project/group runner list. Because you chose the `shell` executor, each job it accepts runs directly on your machine, so you read its variables and `CI_JOB_TOKEN` straight out of the environment. A naming string that matches existing runners keeps it from standing out in the runner list.

## Follow-on

Host access yields the runner token store and cached cloud credentials, which pivot out of GitLab into the connected infrastructure. Captured job variables and job tokens feed [token abuse](token-abuse.md). A reachable registration token here is the payoff of the admin paths in [known admin and API exploits](known-admin-and-api-exploits.md). Container-escape mechanics are covered under the Server container-escape material.

## Tools

- **gitlab-runner** (the official binary) to register and run a rogue runner.
- A request collector for callbacks from job execution.

## References

- [GitLab Runner executors](https://docs.gitlab.com/runner/executors/)
- [GitLab Runner registration](https://docs.gitlab.com/runner/register/)
- [GitLab: security implications of Docker executor privileged mode](https://docs.gitlab.com/runner/executors/docker.html#use-docker-in-docker-with-privileged-mode)
- [GitLab: runner security](https://docs.gitlab.com/runner/security/)
