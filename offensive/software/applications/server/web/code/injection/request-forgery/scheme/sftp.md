---
title: "sftp:// and SSRF: SSH-based file transfer vs generic HTTP fetchers and shared URL parsers"
description: SFTP is usually reached via SSH APIs, not raw sftp:// in generic URL fetchers; note for mislabeled integrations.
keywords:
  - SSRF
  - SFTP
---

# SFTP (SSRF)

Generic `fetch(url)` wrappers rarely implement `sftp://`. Vulnerabilities more often involve **misconfigured** file-transfer jobs that share a URL parser with HTTP SSRF. Map the **actual** library in code review rather than assuming `sftp://` works like `http://`.
