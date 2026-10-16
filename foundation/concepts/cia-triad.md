---
title: "CIA Triad: Confidentiality, Integrity, and Availability"
description: "The three security properties, confidentiality, integrity, and availability, that together define what it means for a system and its data to be secure."
keywords:
  - cia triad
  - confidentiality
  - integrity
  - availability
  - security properties
  - information security
---

# CIA triad

**The CIA triad** names the three properties that security is meant to protect: confidentiality, integrity, and availability. It is the oldest and most widely used model for thinking about what "secure" actually means. Rather than treating security as a single quality, the triad breaks it into three distinct goals, so that a control, a risk, or an incident can be described in terms of which property it affects. When people say a system has been compromised, they almost always mean that one or more of these three properties has failed.

## Three questions to ask of any asset

Each property answers a different question about a piece of information or a system. The [assets](assets-and-attack-surface.md) you protect are worth protecting precisely because a "no" to one of these questions would hurt.

| Property | The question it asks | What a failure looks like | Controls that serve it |
| --- | --- | --- | --- |
| Confidentiality | Can only the right people read it? | A breach exposes customer records | Encryption, access controls, authentication |
| Integrity | Is it accurate, and would we know if it changed? | A bank balance is tampered with, a configuration file is silently edited | Checksums, digital signatures, version history |
| Availability | Can authorized users reach it when they need it? | Ransomware or a denial-of-service attack takes a service offline | Redundancy, backups, capacity planning |

US federal law, as quoted in FIPS 199, gives the formal versions. Confidentiality means "preserving authorized restrictions on information access and disclosure." Integrity means "guarding against improper information modification or destruction," and explicitly includes non-repudiation and authenticity. Availability means "ensuring timely and reliable access to and use of information."

## How each property breaks

The failures are worth studying separately, because they come from different [threats](threats-and-threat-actors.md) and call for different [controls](controls-and-countermeasures.md).

- **Confidentiality breaks by disclosure.** Stolen credentials, a misconfigured storage bucket, an insider copying files, or traffic sent unencrypted over a network. The data is still there and still correct, but someone who should not have it now does, and that cannot be undone.
- **Integrity breaks by unauthorized or undetected change.** An attacker editing records, malware altering a binary, or a bug corrupting a database. The danger is often that nobody notices: a reader must be able to trust that data is what it claims to be, and silent tampering destroys that trust.
- **Availability breaks by denial.** A flood of traffic, encrypted files held for ransom, a deleted backup, or simply a power failure. Security is not only about locking things down: data no one can reach is worthless to the people who depend on it.

A single [vulnerability](vulnerabilities-and-exploits.md) is usually rated by which of these it lets an attacker break, and how badly.

## One ransomware incident, read three ways

Consider a clinic hit by modern "double extortion" ransomware. The attackers first copy patient files to their own servers, then encrypt everything on the clinic's network.

1. **Availability fails first and most visibly.** Staff cannot open schedules or patient charts, and appointments are canceled.
2. **Confidentiality has already failed.** The exfiltrated records are now in criminal hands, and the threat to publish them is the second lever of extortion. Restoring from backup does not fix this.
3. **Integrity is now in doubt.** Even after recovery, the clinic must ask whether any records were altered while the attackers had access. Without trustworthy logs or signed backups, it may not be able to answer.

Describing the incident this way shows why "we restored the systems" is not the same as "we recovered." Each property needs its own response.

## Where the three pull against each other

The goals often conflict, and naming them makes the trade-offs visible.

- Encrypting data and requiring strong authentication raises confidentiality but can reduce availability if a key is lost or a login fails.
- Keeping many backup copies improves availability but creates more places where confidential data might leak.
- Locking records so they cannot be changed protects integrity but can get in the way of legitimate updates.

There is no universal ranking. Medical records demand strong confidentiality and integrity; a public status page demands availability above all, since its content is meant to be seen by everyone. FIPS 199 formalizes this weighting: US federal agencies rate the potential impact of losing each property separately, as low, moderate, or high, and choose their safeguards accordingly.

## Beyond the three letters

The most common mistake is to equate security with confidentiality, keeping secrets secret, and to forget that a corrupted or unreachable system is just as much a security failure. The opposite mistake is to treat the triad as the whole picture. It is a foundation, not a complete framework.

Later models add properties the three letters do not fully capture. **Authenticity** asks whether data or a user is genuine. **Accountability** asks whether actions can be traced to whoever performed them (see [accountability](../principles/accountability.md)). **Non-repudiation** asks whether someone can later deny having sent a message or approved a transaction. Some definitions, including the one in US federal law, fold authenticity and non-repudiation into integrity; others list them as separate goals.

## References

- [NIST SP 800-12 Rev. 1, An Introduction to Information Security](https://csrc.nist.gov/pubs/sp/800/12/r1/final)
- [FIPS 199, Standards for Security Categorization of Federal Information and Information Systems](https://csrc.nist.gov/pubs/fips/199/final)
- [NIST Glossary: Confidentiality](https://csrc.nist.gov/glossary/term/confidentiality)
- [NIST Glossary: Integrity](https://csrc.nist.gov/glossary/term/integrity)
- [ISO/IEC 27000, Information security management systems overview and vocabulary](https://www.iso.org/standard/73906.html)
