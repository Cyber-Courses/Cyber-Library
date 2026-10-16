---
title: "Minimization: Keep Less, Expose Less, Lose Less"
description: "The security principle of reducing data, access, and attack surface to only what a system genuinely requires."
keywords:
  - minimization
  - data minimization
  - attack surface reduction
  - least functionality
  - data retention
  - principle of minimization
---

# Minimization

**Minimization is the discipline of keeping only what you need.** Every record a system stores, every door it leaves open, and every feature it ships is something that can be stolen, misused, or turned against you. The principle says the safest version of each of these is the one that does not exist, so a system should be trimmed to the minimum that still lets it do its job.

## The three things worth minimizing

Minimization is not one rule but the same idea applied to three different kinds of excess. Each has its own owner and its own test.

| What you minimize | The question to ask | What shrinks it |
|---|---|---|
| Data | Do we need to collect and keep this record at all? | Scoped collection, short retention windows, deletion by default |
| Access | Who and what can reach this, and for how long? | Least privilege, scoped credentials, time-bound grants |
| Attack surface | Is this port, service, or feature actually used? | Turning off unused functions, removing dead code and old systems |

The access row is the subject of its own principle. This page focuses on the two that are easy to overlook: the data you hold and the surface you expose.

## "Collect it all" versus "collect what you need"

Data minimization has a legal as well as a security pedigree. Privacy law frames it directly: the European Union's General Data Protection Regulation requires that personal data be "adequate, relevant and limited to what is necessary" for the purpose it was collected for. The United States framework echoes the idea; NIST's privacy guidance treats minimizing what is collected and retained as a core way to reduce risk.

The tension is real. Product and analytics teams have a natural pull toward "collect it all," because data that exists today might be useful tomorrow. Minimization pushes back with a simple accounting: data you never collected cannot leak, and data you deleted on schedule is not in the breach. A retention schedule, where records expire automatically unless there is a reason to keep them, turns that principle into a routine instead of a promise.

## Shrinking the attack surface

Attack surface is the sum of every point where an outsider could try to get in: open network ports, running services, exposed APIs, input fields, installed packages, and user accounts. Minimizing it is often called the principle of least functionality, and NIST SP 800-53 names it directly (control CM-7): configure systems to provide only essential capabilities and disable the rest.

The most dangerous part of the attack surface is usually the part no one remembers. A test server, an old subdomain, or a default admin page that was never turned off sits unmonitored and unpatched, offering a quiet way in.

## Walkthrough: the forgotten subdomain

1. **It gets built.** A team spins up `beta.example.com` for a demo, pointing it at a cloud host and loading it with a copy of production data.
2. **It gets left.** The demo ends. The DNS record stays, the host keeps running an old software version, and no one adds it to the patch or monitoring list.
3. **It gets found.** An attacker scanning the company's domains finds the stale host, exploits the unpatched service, and reaches the sample data, which turned out to be real customer records.
4. **What minimization would have changed.** The data copy should never have existed (data minimization); the host should have been torn down when the demo ended (attack surface); and nothing on it should have held production credentials (least privilege).

## Where minimization reinforces the others

Minimization is the quiet partner of several principles. [Least privilege](least-privilege.md) applies the same logic to permissions, trimming access rather than data or surface. A smaller attack surface gives [defense in depth](defense-in-depth.md) fewer gaps to cover, and it directly supports [zero trust](zero-trust.md), which assumes any exposed service may be probed. Deciding what not to build in the first place is a core move in [security by design](security-by-design.md).

## References

- [GDPR Article 5: Principles relating to processing of personal data](https://gdpr-info.eu/art-5-gdpr/)
- [NIST SP 800-53 Rev. 5 (CM-7 Least Functionality)](https://csrc.nist.gov/pubs/sp/800/53/r5/upd1/final)
- [NIST Privacy Framework](https://www.nist.gov/privacy-framework)
- [OWASP Cheat Sheet: Attack Surface Analysis](https://cheatsheetseries.owasp.org/cheatsheets/Attack_Surface_Analysis_Cheat_Sheet.html)
- [CISA: Secure by Design](https://www.cisa.gov/securebydesign)
