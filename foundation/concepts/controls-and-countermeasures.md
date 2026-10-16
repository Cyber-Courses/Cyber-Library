---
title: "Controls and countermeasures: How defenses reduce risk"
description: "The safeguards that organizations put in place to prevent, detect, and recover from security events, grouped as preventive, detective, and corrective controls."
keywords:
  - security controls
  - countermeasures
  - preventive controls
  - detective controls
  - corrective controls
  - defense in depth
---

# Controls and countermeasures

**A control** is any safeguard or measure put in place to reduce security risk, whether by lowering the chance that something bad happens or by limiting the damage when it does. The term countermeasure is used almost interchangeably, though it often emphasizes a response aimed at a specific threat. Controls are the practical answer to risk: once an organization knows which [assets](assets-and-attack-surface.md) it is protecting and which [threats](threats-and-threat-actors.md) could reach them, controls are the actions it takes to tip the odds in its favor.

## Before, during, after: the timing of a control

The most common way to classify a control is by where it acts on the timeline of a security event.

1. **Preventive controls act before the event** and try to stop it from happening at all. Access permissions, strong authentication, encryption, firewalls, and security awareness training all lower the likelihood that an attack succeeds.
2. **Detective controls act while it unfolds** by revealing that something has happened or is happening. Logging and monitoring, intrusion detection systems, audits, and alerts on unusual activity do not block anything, but they shorten the time an event goes unnoticed.
3. **Corrective controls act afterward** to limit the damage and restore normal operation. Backups and restore procedures, incident response plans, patching, and failover systems belong here.

A single technology can fill more than one role. A backup system is corrective when you restore from it, while the monitoring that confirms each backup actually ran is detective.

## Who or what enforces it

A second classification ignores timing and asks what kind of mechanism carries the control. Crossing the two views gives a simple grid that helps spot empty cells in a security program:

| | Preventive | Detective | Corrective |
|---|---|---|---|
| **Technical (logical)**, enforced by systems | Multi-factor authentication, encryption | Intrusion detection, log monitoring | Automated failover, restore from backup |
| **Administrative**, policies, procedures and training | Acceptable use policy, awareness training | Periodic access reviews, audits | Incident response plan |
| **Physical**, protecting the tangible world | Locks, badge readers | Surveillance cameras | Spare hardware, alternate site |

Most programs need all three rows, because a policy without technical enforcement is easy to ignore, and a locked door does nothing against a stolen password.

## One stolen laptop, several layers

Consider an employee whose laptop is taken from a car. No single control decides the outcome; the layers do.

- **Full-disk encryption** (preventive, technical) keeps the files unreadable to the thief.
- **A password manager and multi-factor authentication** (preventive, technical) mean that saved sessions alone do not open company accounts.
- **A clear reporting procedure** (administrative) gets the loss reported the same day.
- **Sign-in monitoring** (detective, technical) flags any attempt to use the device's credentials from an unfamiliar location.
- **Remote wipe and session revocation** (corrective, technical) cut the device off, and **synchronized backups** (corrective) let the employee resume work on a replacement.

If encryption had been missing, detection and revocation would still limit the harm. This overlap is the idea behind [defense in depth](../principles/defense-in-depth.md): a gap in one layer is covered by another.

## Deciding which controls earn their place

Risk never drops to zero, so the goal is an acceptable level of risk at a reasonable cost, not perfect protection. Choosing controls means weighing how likely and how severe each threat is against what each safeguard costs to build, run, and live with. Controls are largely chosen to close the [weaknesses that exploits target](vulnerabilities-and-exploits.md) and to protect the confidentiality, integrity, and availability described by the [CIA triad](cia-triad.md).

Three beliefs regularly lead programs astray:

- **"More controls always mean more security."** Overlapping or poorly chosen controls add cost and complexity without reducing meaningful risk, and they can frustrate users into working around them.
- **"Prevention is enough."** Prevention eventually fails, so detection and recovery are what keep a failure from becoming a disaster.
- **"Once deployed, a control keeps working."** Controls decay as systems, threats, and staff change. They must be tested, reviewed, and updated to stay effective, which is why NIST pairs its SP 800-53 control catalog with a companion guide, SP 800-53A, on how to assess each control.

## References

- [NIST Glossary: Security Control](https://csrc.nist.gov/glossary/term/security_control)
- [NIST Glossary: Countermeasures](https://csrc.nist.gov/glossary/term/countermeasures)
- [NIST SP 800-53 Rev. 5, Security and Privacy Controls for Information Systems and Organizations](https://csrc.nist.gov/pubs/sp/800/53/r5/upd1/final)
- [NIST Cybersecurity Framework](https://www.nist.gov/cyberframework)
- [ISO/IEC 27002, Information Security Controls](https://www.iso.org/standard/75652.html)
- [CISA: Cyber Essentials](https://www.cisa.gov/resources-tools/resources/cyber-essentials)
