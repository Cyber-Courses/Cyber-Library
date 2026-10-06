---
title: "Git daemon: cloning private repositories over unauthenticated git://"
description: "Abusing an unauthenticated Git daemon: how the git:// protocol on port 9418 serves repositories with no authentication, enumerating and cloning exposed repositories, the export-ok and exportAll gates, and the danger of an enabled receive-pack or upload-archive service."
keywords:
  - git daemon
  - git protocol
  - port 9418
  - anonymous clone
  - export-ok
---

# Daemon

`git daemon` exposes repositories over the `git://` protocol on TCP 9418 with no authentication and no transport encryption by design: it is meant for anonymous read of public mirrors. When it is pointed at an internal or private repository tree, anyone who can reach the port can clone the code and its full history.

## Fingerprint and enumerate

```bash
nmap -p9418 -sV <target>
# the daemon only serves a repo that is explicitly exported, so test specific paths
git ls-remote git://<target>/project.git        # refs listed => the repo is served
```

A repository is served only when it is marked exportable: a `git-daemon-export-ok` file in the bare repo, or the daemon started with `--export-all` (which exports every repository under its base path regardless of the marker). An `--export-all` daemon is the common misconfiguration, it turns the whole base directory into anonymous-readable repositories.

## Clone and loot

```bash
git clone git://<target>/project.git
cd project && git log -p --all          # then mine it as in secrets-in-history
```

There is no authentication step, so a successful clone is immediate source and history disclosure. If you do not know the repository names, try common ones (`.git`, `project.git`, the app name) and scrape any CI or deployment config you already have for `git://` URLs.

## Writable and extended services

The daemon's service set matters. By default only `upload-pack` (clone/fetch) is enabled, but a daemon started with `--enable=receive-pack` accepts anonymous pushes, letting you write to the repository, which combined with server-side hooks is code execution (see [hooks and config execution](hooks-and-config-execution.md)). An enabled `upload-archive` service has historically allowed reaching unintended objects. Check what the daemon answers:

```bash
# anonymous push is accepted only if receive-pack is enabled for the repo
git push git://<target>/project.git HEAD:refs/heads/test
```

A push that is accepted rather than refused is a writable anonymous repository.

## References

- [git-daemon documentation](https://git-scm.com/docs/git-daemon)
- [Git protocol (Pro Git): the dumb and smart protocols](https://git-scm.com/book/en/v2/Git-Internals-Transfer-Protocols)
