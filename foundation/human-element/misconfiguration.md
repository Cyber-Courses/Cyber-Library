---
title: "Misconfiguration: When Secure Software Is Deployed Insecurely"
description: "The gap between a secure design and a secure deployment, where default settings and small oversights quietly open the door to attackers."
keywords:
  - misconfiguration
  - security misconfiguration
  - default settings
  - hardening
  - secure defaults
  - configuration management
---

# Misconfiguration

**Misconfiguration** is the gap between a system that could be secure and one that actually is, created when software is deployed with weak defaults, unnecessary features, or settings that were never tightened. The code may be sound and the design may be strong, yet the running system still exposes data or access because someone left a setting at its factory value, skipped a step, or did not understand what an option controlled. It is consistently one of the most common root causes of breaches, and it sits largely on the human side of security because it stems from deployment decisions and oversights. A vendor that ships an unsafe default has made a [design error](design-and-development-errors.md) affecting every installation; misconfiguration is the operator accepting that default, or drifting away from a secure baseline, in a specific deployment.

## Sound code, unsafe settings

A product can pass every code review and still be shipped into an unsafe state. That is the line separating misconfiguration from [design and development errors](design-and-development-errors.md): with a design error, the fix is a new version of the software; with a misconfiguration, the fix is a change in how the existing software is set up.

The gap is wide because modern systems have thousands of tunable settings across operating systems, databases, cloud services, and applications, and a single wrong value can undo otherwise good security. It also hides well. A misconfigured system usually works fine for normal users, so nobody complains, and the problem stays invisible until someone malicious goes looking for it.

## Four ways a setting ends up wrong

Most bad settings arrive by one of these routes:

1. **It was never changed.** The vendor shipped it open or with a known password, and the installer accepted the defaults.
2. **It was loosened to make something work.** A permission was widened or a check disabled during troubleshooting or a deadline, and never restored.
3. **Nobody understood what it did.** An option with an unclear name or a buried side effect was set without knowing its security impact.
4. **It drifted.** The system was hardened once, then manual changes, upgrades, and new components slowly moved it away from that baseline.

The first route is the reason CISA's Secure by Design initiative pushes vendors to ship products locked down by default, so the safe state is the one customers get without doing anything.

## What the wrong value exposes

| Setting left wrong | What an attacker gets |
|---|---|
| Default credentials (admin/admin, vendor-shipped passwords) | A direct login to an admin console or device |
| Public storage buckets, databases, or file shares | Bulk download of data, often found by automated scanners |
| Sample apps, debug modes, directory listing, unused services | Extra code paths and files that widen the [attack surface](../concepts/assets-and-attack-surface.md) |
| Encryption available but not turned on | Data readable in transit or at rest |
| Verbose error messages and stack traces | A map of internal components and their versions |
| Insecure legacy settings kept for backward compatibility | Old, weaker protocols and options an attacker can downgrade to |
| Accounts with more rights than their job needs | A small compromise that turns into a large one (see [least privilege](../principles/least-privilege.md)) |

None of these needs special tooling to exploit. That is why misconfiguration ranks so high in industry data: in the OWASP Top 10:2025, Security Misconfiguration moved up from fifth place to second, and OWASP reports that every application it tested had some form of it.

## The view from red and blue teams

In 2023, NSA and CISA published the ten misconfigurations their red and blue teams found most often in large organizations. The list opens with default configurations of software and applications, followed by improper separation of user and administrator privilege, and also includes insufficient internal network monitoring, lack of network segmentation, poor patch management, weak or misconfigured MFA, insufficient access controls on network shares, and poor credential hygiene.

Two lessons stand out. First, these are not exotic weaknesses: they are the routine settings that busy teams leave for later. Second, the cloud makes them scale badly, since one bad template or default can be copied across hundreds of resources in seconds, turning a single oversight into an organization-wide exposure.

## Keeping a configuration honest over time

Hardening is not a one-time task, because configurations drift. A durable approach treats settings as something to define, check, and prune continuously:

- **Start locked down.** Prefer systems that ship with secure defaults, and treat every loosened setting as a deliberate, documented choice. This is the configuration side of [security by design](../principles/security-by-design.md).
- **Measure against a baseline.** Apply recognized hardening guides, such as the CIS Benchmarks or NIST SP 800-123 for servers, rather than relying on memory.
- **Define settings as code.** Deploy from reviewed templates so the same checked baseline is applied every time, and a fix to the template fixes every copy.
- **Review changes with a second person.** Separation of duties catches honest mistakes before they reach production.
- **Scan for drift.** Configuration and posture scanning surfaces risky settings and unapproved changes before attackers find them.
- **Remove what you do not use.** Disabling unused features, accounts, and services shrinks the attack surface and the number of settings that can go wrong.

Tools only go so far: administrators still need to recognize a dangerous option when they see one, which is where [training and awareness](training-and-awareness.md) comes in. Like the everyday shortcuts described in [user behavior and misuse](user-behavior-and-misuse.md), misconfiguration creates openings without any software bug, and a single bad setting can quietly undermine the confidentiality, integrity, and availability described in the [CIA triad](../concepts/cia-triad.md).

## References

- [OWASP Top 10:2025, A02 Security Misconfiguration](https://owasp.org/Top10/2025/A02_2025-Security_Misconfiguration/)
- [NSA and CISA: Top Ten Cybersecurity Misconfigurations (AA23-278A)](https://www.cisa.gov/news-events/cybersecurity-advisories/aa23-278a)
- [MITRE CWE-16: Configuration](https://cwe.mitre.org/data/definitions/16.html)
- [NIST SP 800-123, Guide to General Server Security](https://csrc.nist.gov/pubs/sp/800/123/final)
- [CIS Benchmarks](https://www.cisecurity.org/cis-benchmarks)
- [CISA: Secure by Design](https://www.cisa.gov/securebydesign)
