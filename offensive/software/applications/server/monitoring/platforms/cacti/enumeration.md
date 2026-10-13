---
title: "Enumeration: fingerprinting Cacti"
order: 4
description: "Cacti is identified by its web interface under /cacti/ and its version string in the page and changelog, and the version is decisive because its remote-code-execution vulnerabilities are version-specific. Reading the exact build directs the choice of exploit, including whether an unauthenticated command-injection path is present."
keywords:
  - cacti enumeration
  - version
  - fingerprint
  - cacti path
  - advisory
---

# Enumeration

Cacti's web interface sits under `/cacti/` and exposes the product and version in the login page, the footer, and the bundled `CHANGELOG`. The version is the decisive fact, because Cacti's command-injection and SQL-injection vulnerabilities, including unauthenticated ones, are specific to particular builds. Reading the exact version first tells you which exploit chain applies and whether an unauthenticated remote-code-execution path (such as the one in the remote-agent handling) is present on this instance.

```bash
curl -sk https://<target>/cacti/ | grep -ioE 'Version [0-9.]+'
curl -sk https://<target>/cacti/CHANGELOG | head                 # often readable, exact version
curl -sk https://<target>/cacti/include/cacti_version            # version file in some builds
```

## Exploitation notes

- The version maps directly to the applicable [known exploits](known-exploits.md); Cacti's RCE bugs are version-specific, so fingerprint precisely (the `CHANGELOG` or version file gives it exactly).
- Determine whether an unauthenticated path applies to this build (some command-injection flaws need no login), which changes whether you need [credentials](authentication.md) first.
- The `/cacti/` path and readable `CHANGELOG`/version file make fingerprinting easy and unauthenticated.
- Route to the exploit chains or, with access, to [credential harvesting](credential-harvesting.md).

## References

- [Cacti releases](https://github.com/Cacti/cacti/releases)
- [Cacti security advisories](https://github.com/Cacti/cacti/security/advisories)
