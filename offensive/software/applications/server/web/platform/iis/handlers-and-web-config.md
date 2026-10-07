---
title: "IIS handlers and web.config: execution mappings, source disclosure, and config abuse"
order: 3
description: "Exploiting IIS handler mappings and web.config: extension handlers that execute uploads, source disclosure from missing handlers, and per-directory web.config upload abuse."
keywords:
  - iis handlers
  - web.config
  - handler mapping
  - upload rce
  - source disclosure
---

# Handlers and web.config

IIS maps request extensions to handlers (the ASP.NET pipeline, the static file handler, CGI/FastCGI) and reads a per-directory `web.config` that can change those mappings. Misconfigurations here execute uploads, disclose source, or let an attacker reconfigure a directory they can write to.

## Handler mapping and upload execution

If a directory that accepts uploads is served by a handler that executes the uploaded extension, the upload is RCE. The IIS-specific extensions to get past denylists include:

```
shell.aspx   shell.asp   shell.ashx   shell.asmx   shell.cer   shell.soap
```

`.cer` and `.asa`/`.asax` have historically been mapped to executing handlers on misconfigured servers. Combine with the [NTFS filename tricks](ntfs-filename-tricks.md) (`shell.aspx;.jpg`, trailing dot) to slip an executable extension past an upload filter while IIS still runs it. The `;` semicolon trick (`file.asp;.jpg`) made older IIS execute the part before the semicolon.

## web.config as an execution vector

`web.config` is read per directory. Where uploads land in a directory without a parent lock and the application does not block it, uploading a crafted `web.config` can:

- register a handler that executes a benign extension in that directory, or
- define an ASP-classic block or an `httpHandlers`/`handlers` entry that runs code,
- and in some setups the `web.config` itself executes server-side includes or expression syntax when processed.

This turns "upload any file" into RCE even when script extensions are blocked, analogous to the Apache `.htaccess` upload.

## Source disclosure

- Request executable scripts via `::$DATA` / trailing dot (see [NTFS filename tricks](ntfs-filename-tricks.md)) to get raw source instead of execution.
- A directory where the ASP.NET handler is not wired (static-file handler serving `.aspx`) returns source; this appears after misconfigured deployments or when a handler is removed.
- `web.config` itself, if served as static content (handler misconfigured), discloses connection strings, machine keys, and handler layout. A leaked `machineKey` enables [ViewState forgery](../../runtime/insecure-deserialization/dotnet-deserialization.md).

## Exploitation

- Fuzz uploads with IIS executable extensions plus filename tricks; confirm execution with a benign marker.
- Where uploads allow it, try planting a `web.config` to remap or execute.
- Probe known `.aspx`/`.asmx` for `::$DATA` source, and request `/web.config` (and `::$DATA`) for config disclosure and the machine key.

## Tools

- Burp (upload fuzzing); **IIS-ShortName-Scanner** to find hidden scripts; content-discovery tooling.

## References

- Microsoft IIS: handler mappings, web.config, request filtering
- OWASP: Unrestricted file upload
