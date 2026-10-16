---
title: "CIA Triad: The Three Properties of a Secure System"
description: "Confidentiality, integrity, and availability are the three properties that together define what it means for a system and its data to be secure."
keywords:
  - cia triad
  - confidentiality
  - integrity
  - availability
  - security properties
  - dad triad
---

# CIA triad

**The CIA triad** splits the vague goal of "security" into three concrete properties worth protecting: confidentiality, integrity, and availability. Its value is that it gives every control, risk, and incident a precise label. Instead of saying a system was "hacked," you can say which of the three properties failed, and that single distinction usually decides how you respond.

## Confidentiality, integrity, and availability, one at a time

Each letter answers a different question about a piece of data or a service. A property only matters because a "no" to its question would cause real harm to the [assets](assets-and-attack-surface.md) you depend on.

| Property | The question it answers | Served by |
| --- | --- | --- |
| Confidentiality | Can only authorized people read it? | Encryption, access control, authentication |
| Integrity | Is it accurate, and would we notice a change? | Message authentication codes (MACs), digital signatures, change logs |
| Availability | Can the right people reach it when needed? | Redundancy, backups, capacity planning |

The three are independent: a system can be perfectly confidential yet constantly offline, or highly available yet full of silently altered records.

## The DAD triad: how each property fails

Every property has a matching failure, and together they form what is sometimes called the DAD triad. Naming the failure points you toward the [controls](controls-and-countermeasures.md) that prevent it.

- **Disclosure** breaks confidentiality. Stolen credentials, a public storage bucket, or traffic sent in the clear. The data is unchanged and still present, but someone who should not see it now does, and that cannot be taken back.
- **Alteration** breaks integrity. An attacker edits a record, malware patches a binary, or a bug corrupts a table. The real danger is the version nobody notices.
- **Destruction** (or denial) breaks availability. Ransomware encryption, a flood of traffic, a deleted backup, or a power cut. Data no one can reach is worthless to the people who need it. Modern ransomware often also breaks confidentiality, exfiltrating the data before encrypting it so operators can threaten to leak it.

Most [vulnerabilities](vulnerabilities-and-exploits.md) are scored by which of these three they let an attacker achieve, and how severely.

## One build server, three ways to abuse it

Picture an attacker who gains access to a company's software build server. A single foothold threatens all three properties at once:

1. **Against confidentiality,** they copy the source code and the signing keys stored alongside it.
2. **Against integrity,** they slip a backdoor into the next release, which ships to every customer carrying a valid signature.
3. **Against availability,** they wipe the pipeline on the way out, halting all future builds.

The same intrusion reads as a leak, a tampering, and an outage depending on which property you examine. That is why "we rebuilt the server" rarely means the incident is over.

## When the three properties pull apart

The goals routinely conflict, and the triad makes those trade-offs explicit rather than accidental. Strong encryption raises confidentiality but can destroy availability if a key is lost. Extra backup copies raise availability but multiply the places confidential data can leak. There is no fixed ranking: medical records weight confidentiality and integrity highest, while a public status page lives or dies by availability. Layering safeguards, as in [defense in depth](../principles/defense-in-depth.md), is how teams serve more than one property without betting everything on a single measure.

## What the three letters leave out

The triad is a foundation, not a complete model. It says little about **authenticity** (is this data or user genuine?), **non-repudiation** (can someone later deny an action?), or [accountability](../principles/accountability.md) (can an action be traced to whoever took it?). Some standards fold these into integrity; others list them separately. Treat CIA as the first vocabulary you reach for, not the last.

## References

- [NIST SP 800-12 Rev. 1, An Introduction to Information Security](https://csrc.nist.gov/pubs/sp/800/12/r1/final)
- [FIPS 199, Standards for Security Categorization of Federal Information and Information Systems](https://csrc.nist.gov/pubs/fips/199/final)
- [NIST Glossary: Confidentiality](https://csrc.nist.gov/glossary/term/confidentiality)
- [ISO/IEC 27000, Information security management systems overview and vocabulary](https://www.iso.org/standard/73906.html)
