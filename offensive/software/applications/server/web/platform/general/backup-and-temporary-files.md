---
title: "Backup and temporary files: editor swaps, .bak, and leftover archives"
description: "Recovering source and secrets from backup and temporary files left in the web root: editor swap files, .bak/~/.old copies, and deployment archives the server serves as static content."
keywords:
  - backup files
  - .bak
  - vim swap
  - source disclosure
  - temporary files
---

# Backup and temporary files

Editing files on the server, saving "just in case" copies, or leaving a deployment archive in the web root produces files the server treats as static (so it serves their raw bytes instead of executing them). That turns a `.php` into readable source and an archive into the whole application.

## What to look for

For a known script `app.php`, probe the common backup and swap variants:

```
app.php.bak      app.php~        app.php.old      app.php.save
app.php.orig     app.php.swp     app.php.swo      app.php.1
.app.php.swp     app.php.txt     app.php.copy     app.php.dist
#app.php#        app.phps
```

Editor artifacts are especially reliable: Vim writes `.<name>.swp`/`.swo`, Emacs writes `name~` and `#name#`, and a crashed edit leaves a recoverable swap. `.phps` is a source-highlight handler that returns PHP as source. Any of these served with a `text/plain` content type is raw source.

Also sweep for deployment leftovers in the root and common paths:

```
backup.zip   site.tar.gz   www.zip   backup.sql   db.sql   dump.sql
release.tar  app.tar.gz    .env.bak  config.php.bak
```

## Finding them

Content discovery with a wordlist is the practical approach, mutating known filenames with backup suffixes:

```bash
ffuf -w paths.txt -u https://target/FUZZ -mc 200
# backup-suffix mutation of discovered files:
for f in app.php index.php config.php; do
  for s in .bak '~' .old .swp .save .orig; do echo "$f$s"; done
done > bak.txt
ffuf -w bak.txt -u https://target/FUZZ -mc 200
```

Recover a Vim swap with `vim -r` offline:

```bash
curl -s https://target/.app.php.swp -o app.swp && vim -r app.swp
```

## Exploitation

- Raw source from a `.bak`/swap gives the same leverage as a VCS dump: sinks, secret keys, credentials.
- A `.sql` dump or archive can contain the full database (password hashes, tokens) or the entire app including config.
- Combine with directory listing and VCS exposure; one often points at the others (a listing reveals the backup name, a `.git` history reveals which files exist to probe for backups).

## Tools

- **ffuf** / **feroxbuster** with backup-suffix wordlists; **vim -r** for swap recovery.

## References

- OWASP WSTG: Review old backup and unreferenced files
- SecLists: backup-file wordlists
