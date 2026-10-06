---
title: "Secrets in history: mining Git commits, objects, and the reflog"
description: "Recovering credentials from a Git repository's history: why deleting a secret from the working tree leaves it in the object store, searching all commits with the pickaxe, recovering dangling and force-pushed objects with fsck and the reflog, and automating with TruffleHog and Gitleaks."
keywords:
  - secrets in git history
  - git pickaxe
  - dangling objects
  - trufflehog
  - gitleaks
---

# Secrets in history

Removing a secret in a later commit does not remove it from the repository. Git keeps every version of every file as an immutable object keyed by its content hash, so a key, token, or password that was ever committed stays reachable through history, through objects no branch points to any more, and through the reflog, as long as you hold the repository. Any repository you have cloned or recovered (a dumped `.git`, an anonymous clone) carries that full object store with it.

## Search the committed history

The pickaxe (`-S`/`-G`) finds the exact commit that introduced or removed a string, across every ref, which is where "I deleted the password in the next commit" cleanups fall down:

```bash
git log -p --all -S 'password'                 # commits that add/remove the literal "password"
git log -p --all -G 'AKIA[0-9A-Z]{16}'         # regex: an AWS access-key id anywhere in a diff
git log --all --oneline -- config/secrets.yml  # every revision of a sensitive path, then git show each
```

`--all` walks all refs rather than just the current branch, and `--source` names the ref each hit lives on.

## Recover dangling and force-pushed objects

Commits removed by `git rebase`, `git commit --amend`, or a force-push become unreferenced but remain in the object database of a repository you already hold. `git fsck` surfaces them, and the local reflog records where branches used to point:

```bash
git fsck --lost-found --dangling 2>/dev/null   # dangling commit/blob SHAs in this repo
git cat-file -p <dangling-sha>                 # read the content directly
git reflog --all                               # SHAs branches pointed to before a reset/amend/force-push
```

A dangling blob that `git log` cannot reach is exactly where a rewritten "fix: remove secret" commit left the original value.

## Automate it

Run a dedicated scanner over the full history rather than grepping blind; they carry hundreds of credential patterns and can verify live keys:

```bash
trufflehog git file://./loot --only-verified    # scan a local clone/dump, confirm which creds still work
gitleaks detect --source . --log-opts '--all'   # regex + entropy across all refs
```

## Follow-on

A recovered credential is immediate reuse material: database and cloud keys, CI tokens, SSH deploy keys, and API secrets. Validate it, then pivot to whatever it unlocks. Keys that were "rotated" after the leak are worth testing anyway, rotation is frequently incomplete.

## Tools

- [TruffleHog](https://github.com/trufflesecurity/trufflehog)
- [Gitleaks](https://github.com/gitleaks/gitleaks)

## References

- [git log pickaxe (-S/-G)](https://git-scm.com/docs/git-log)
- [git fsck](https://git-scm.com/docs/git-fsck)
