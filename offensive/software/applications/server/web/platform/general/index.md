---
title: "General server exposure: source, secrets, and deployment detail on any server"
order: 1
description: "Product-agnostic platform exposures that apply whatever the web server: status endpoints, version-control directories, backup and temporary files, directory listings, and leaked config files."
keywords:
  - information disclosure
  - server-status
  - git exposure
  - backup files
  - directory listing
---

# General

Before fingerprinting matters, every server can be made to leak. This section holds the exposures that are **product-agnostic**: they depend on files and endpoints being reachable, not on which web server is running. Checking these first is cheap and high-yield, and keeps the per-product sections free of repetition.

## Why it matters

This is reconnaissance that pays immediately. Recovered source reveals the exact sinks for injection and deserialization; a leaked `.env` or config file hands over database and API credentials and framework secret keys (which unlock signed cookies and ViewState); a `server-status` page exposes other users' requests and internal paths. None of it requires touching the application code.

## Pages

- **[Status and info endpoints](status-and-info-endpoints.md)**: `server-status`, `server-info`, nginx `stub_status`, and PHP-FPM status pages.
- **[Version control directories](version-control-directories.md)**: exposed `.git`, `.svn`, and `.hg`, and reconstructing source from them.
- **[Backup and temporary files](backup-and-temporary-files.md)**: editor swap files, `.bak`/`~`/`.old`, and leftover archives.
- **[Directory listing](directory-listing.md)**: autoindex and directory browsing as an enumeration source.
- **[Exposed config and dotfiles](exposed-config-and-dotfiles.md)**: `.env`, config files, `.htaccess`, and metadata dotfiles.

## References

- PortSwigger Web Security Academy: Information disclosure
- OWASP WSTG: Testing for information leakage
