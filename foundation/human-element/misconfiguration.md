---
title: "Misconfiguration: when secure software is deployed insecurely"
description: "Misconfiguration is the gap between a secure design and a secure deployment, where default settings and small oversights quietly create openings for attackers."
keywords:
  - misconfiguration
  - security misconfiguration
  - default settings
  - secure defaults
  - configuration drift
  - hardening baseline
---

# Misconfiguration

**Misconfiguration** is the distance between a system that could be secure and one that actually is, opened up when software is deployed with weak defaults, unused features left on, or settings that were never tightened. The code can be sound and the design can be strong, yet the running system still leaks data or hands out access because someone kept a factory value, skipped a hardening step, or did not realize what an option controlled. It is one of the most common root causes of real breaches, and it belongs to the human side of security because it grows out of deployment choices rather than flaws in the software itself.

## Code that is fine, settings that are not

The clearest way to understand misconfiguration is by what it is not. A [design and development error](design-and-development-errors.md) lives in the software, so the fix is a new version. A misconfiguration lives in how an existing, correct product was set up, so the fix is a changed setting, not a patch.

The gap is wide for two reasons. Modern systems expose thousands of tunable options across operating systems, databases, cloud services, and applications, and one wrong value can undo everything else. The problem also hides well: a misconfigured system usually works fine for ordinary users, so nobody files a complaint, and the opening stays invisible until someone goes looking for it.

## How a setting ends up wrong

Most bad settings arrive by one of four routes:

1. **It was never changed.** The product shipped open, or with a known password, and whoever installed it accepted the defaults.
2. **It was loosened to make something work.** A permission was widened or a check switched off during troubleshooting, then never restored.
3. **Nobody understood it.** An option with an unclear name or a buried side effect was set without anyone knowing its security impact.
4. **It drifted.** A system hardened once slowly moved away from that baseline as upgrades, manual fixes, and new components piled up.

## What a loose setting hands over

The cost of a wrong value is concrete. A few common cases:

| Setting left wrong | What an attacker gains |
|---|---|
| Default or vendor-shipped credentials | A direct login to an admin console |
| Public storage buckets or databases | Bulk download of data, often found by scanners |
| Debug mode, sample apps, directory listing | Extra code paths that widen the [attack surface](../concepts/assets-and-attack-surface.md) |
| Verbose error messages | Internal paths, versions, and stack traces that aid the next step |

## Secure by default and who carries the burden

Route one, the unchanged default, is why CISA's Secure by Design initiative asks vendors to ship products locked down out of the box, so the safe state is the one customers get for free. That reframes responsibility: when the default is unsafe, the vendor has made a design decision that burdens every installation. When the default is safe, misconfiguration becomes a narrower matter of operators drifting away from it, which pairs naturally with [security by design](../principles/security-by-design.md).

## Where it meets the other human failures

Misconfiguration rarely stands alone. It often follows [user behavior and misuse](user-behavior-and-misuse.md), as when a rushed workaround quietly loosens a control, and it is blunted by [training and awareness](training-and-awareness.md) that teaches operators to question a default instead of accepting it.

## References

- [OWASP Top 10: A05:2021 Security Misconfiguration](https://owasp.org/Top10/A05_2021-Security_Misconfiguration/)
- [NIST SP 800-53 Rev. 5 (Configuration Management family)](https://csrc.nist.gov/pubs/sp/800/53/r5/upd1/final)
- [CISA Secure by Design](https://www.cisa.gov/securebydesign)
- [CWE-1188: Initialization of a Resource with an Insecure Default](https://cwe.mitre.org/data/definitions/1188.html)
- [NIST SP 800-123: Guide to General Server Security](https://csrc.nist.gov/pubs/sp/800/123/final)
