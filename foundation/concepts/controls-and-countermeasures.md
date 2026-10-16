---
title: "Controls and Countermeasures: Preventive, Detective, and Corrective"
description: "How preventive, detective, and corrective controls work together to reduce risk, and the different forms a single safeguard can take."
keywords:
  - security controls
  - countermeasures
  - preventive controls
  - detective controls
  - corrective controls
  - compensating controls
---

# Controls and countermeasures

**A control** is any safeguard or measure put in place to reduce risk to an asset, and a **countermeasure** is the same idea named from the attacker's side: a response aimed at a specific threat. The two words are used almost interchangeably. What matters is the job they do. A control does not make a weakness disappear; it lowers the chance that the weakness is exploited, the damage if it is, or the time before anyone notices. Security is the practice of choosing a sensible set of these measures and arranging them so they reinforce each other.

## Prevent, detect, correct

The most useful way to sort controls is by when they act relative to an incident. Each function answers a different question, and a serious defense needs all three.

| Function | Question it answers | Examples |
| --- | --- | --- |
| Preventive | How do we stop it from happening? | Access controls, encryption, firewalls, input validation, staff training |
| Detective | How do we know it is happening or has happened? | Logging, intrusion detection, file-integrity monitoring, audits |
| Corrective | How do we recover once it has? | Backups and restores, patching, incident response, isolating a host |

A fourth function, **deterrent**, discourages an actor before they try (visible cameras, warning banners, the prospect of prosecution). Prevention is never perfect, which is exactly why detective and corrective controls exist: they assume some attacks will get through and shorten the time an attacker stays unnoticed and unchecked.

## Administrative, technical, physical

The function above says what a control does; a second axis says how it is built. The same goal can be met in any of these forms, and mature programs combine them rather than leaning on one.

- **Administrative** controls are policies, procedures, and training: the rules people are expected to follow, such as an access-review policy or a clean-desk rule.
- **Technical** (or logical) controls are enforced by hardware or software: a password requirement, disk encryption, a network segmentation rule.
- **Physical** controls act on the tangible world: locked doors, badge readers, cable locks, secure disposal of drives.

Locking a server room (physical), requiring a login (technical), and forbidding shared accounts in policy (administrative) can all serve the same aim of keeping the wrong people away from a machine.

## When the primary control cannot be used

Sometimes the recommended control is impossible, too costly, or breaks something. A **compensating control** is a deliberate substitute that provides comparable protection by another route. If an old system cannot support multi-factor login, placing it on an isolated network segment and watching its logs closely can cover much of the same risk. Compensating controls are legitimate, but they are a trade, not a free pass: each one should be documented, and its weaker coverage understood, rather than quietly accepted.

## A stolen laptop, control by control

Picture an employee's laptop taken from a car. Follow the controls in order:

1. **Preventive.** Full-disk encryption means that while the laptop stays powered off, the thief sees only ciphertext, so the data stays confidential even though the hardware is gone.
2. **Detective.** The employee reports the loss and device-management logs confirm the laptop has not checked in, flagging it as missing.
3. **Corrective.** Administrators revoke its credentials and issue a remote wipe, and the user restores their files from backup onto a replacement.

No single measure handled the incident. The encryption bought time, the reporting and logs caught it, and the wipe and restore closed it out.

## How controls connect to risk and defense

Controls exist to protect the confidentiality, integrity, and availability named in the [CIA triad](cia-triad.md), by shrinking the opportunity available to the [threats and threat actors](threats-and-threat-actors.md) that target an organization. Each control addresses one or more [vulnerabilities and exploits](vulnerabilities-and-exploits.md), and layering independent controls so that one failing does not expose everything is the principle of [defense in depth](../principles/defense-in-depth.md).

## References

- [NIST SP 800-53 Rev. 5, Security and Privacy Controls for Information Systems and Organizations](https://csrc.nist.gov/pubs/sp/800/53/r5/upd1/final)
- [NIST Glossary: Security Control](https://csrc.nist.gov/glossary/term/security_control)
- [NIST Glossary: Countermeasures](https://csrc.nist.gov/glossary/term/countermeasures)
- [CIS Critical Security Controls](https://www.cisecurity.org/controls)
- [ISO/IEC 27002, Information security controls](https://www.iso.org/standard/75652.html)
