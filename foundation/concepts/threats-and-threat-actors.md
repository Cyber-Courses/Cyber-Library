---
title: "Threats and Threat Actors: Who Attacks and Why"
description: "What a threat is in security terms, the kinds of actors behind threats, and the motivations and capabilities that shape how they behave."
keywords:
  - threats
  - threat actors
  - threat modeling
  - adversary motivation
  - attacker capabilities
  - cyber threat intelligence
---

# Threats and threat actors

**A threat** is any circumstance or event with the potential to harm an asset by exploiting a vulnerability. A **threat actor** is the person, group, or entity that carries out, or intends to carry out, that harm. Keeping these two ideas distinct is useful: a threat describes what could go wrong, while the actor describes who would make it happen and why. Together they help you reason about not just whether a weakness exists, but whether anyone is motivated and able to use it.

## A threat needs something to harm and a way in

A threat only matters in relation to something worth protecting (an [asset](assets-and-attack-surface.md)) and a weakness that lets harm occur (a [vulnerability](vulnerabilities-and-exploits.md)). Remove either one and the threat has nothing to act on.

Two examples show the chain:

1. **Flooding and a data center.** The servers inside have value, and the building can take on water. The flood is the threat, the servers are the asset, the low-lying site is the weakness.
2. **Phishing and an employee account.** People can be tricked, and an account, once stolen, grants access to something useful. The phishing email is the threat, the account and what it unlocks are the asset, human trust is the weakness.

What the threat ultimately puts at risk is the confidentiality, integrity, or availability of that asset, as described in the [CIA triad](cia-triad.md).

## Threats with no adversary behind them

Not every threat comes from a hostile actor. Security distinguishes between **deliberate** threats, such as an intruder, and **accidental or environmental** ones: a power outage, a hardware failure, a storm, or an honest mistake by an employee. NIST's risk assessment guidance treats these as separate threat source categories alongside adversarial ones, because they call for different questions. An adversary adapts to your defenses; a failing disk does not. The rest of this page focuses on the deliberate side, where a threat actor is involved.

## Five families of threat actors

Threat actors are commonly grouped by who they are and what drives them. The boundaries blur in practice (a state may tolerate or hire criminals, an insider may be recruited by an outsider), but the families are a useful starting point.

| Actor | Typical motivation | Typical capability | What it often looks like |
|---|---|---|---|
| Cybercriminals | Money | Moderate to high, often bought or rented | Ransomware, fraud, theft of data they can sell |
| Nation-state actors | Espionage, sabotage, strategic advantage for a government | High: funding, time, custom tooling | Long, quiet intrusions into government, defense, and critical infrastructure |
| Hacktivists | A political or social cause | Low to moderate | Website defacement, leaks, disruption timed to events |
| Malicious insiders | Grievance, money, or coercion | Legitimate access, which often outweighs skill | Misuse of access by current or former employees and contractors |
| Thrill seekers and hobbyists | Curiosity, reputation, the challenge itself | Varies widely | Probing exposed systems, publicized one-off intrusions |

Public catalogs such as MITRE ATT&CK's Groups list track specific named actors within these families and the techniques each has been observed using.

## Motivation, capability, opportunity

Two dimensions describe any actor:

- **Motivation** is why they act: money, information, disruption, ideology, or ego.
- **Capability** is what they can actually do, shaped by skill, funding, time, and tools.

A highly motivated but unskilled actor is a different problem than a patient, well-funded one. Analysts often add a third factor, **opportunity**: the access or exposure an actor needs to act at all.

Reducing any of the three lowers risk, but defenders rarely influence the first two. You cannot make a criminal less interested in money or a government less well funded. Opportunity is different: patching, reducing exposed services, and limiting access through [controls and countermeasures](controls-and-countermeasures.md) all shrink it directly.

## One exposed server, three different attackers

Picture a small company running an unpatched, internet-facing file server. Three actors could reach it, and each corrects a common assumption.

- **The automated scanner.** A criminal group's tooling sweeps the internet for that exact flaw and drops ransomware on every match. Nobody chose this company; it was simply reachable and unpatched. "We are too small to be a target" fails because much crime is opportunistic and automated.
- **The organized crew.** The access is sold by one specialist to another who handles extortion, while a third negotiates payment. Most threat actors are not lone geniuses but organized groups with specialized roles, shared tools, and even support services.
- **The departing administrator.** A contractor whose account was never disabled copies the server's contents before leaving. Insiders, and plain accidents, account for a large share of real incidents, so focusing only on outsiders leaves a gap.

The same weakness, three very different actors: understanding who might use a vulnerability, and why, is what turns a list of flaws into a prioritized risk picture.

## References

- [NIST Glossary: Threat](https://csrc.nist.gov/glossary/term/threat)
- [NIST Glossary: Threat Actor](https://csrc.nist.gov/glossary/term/threat_actor)
- [NIST SP 800-30 Rev. 1, Guide for Conducting Risk Assessments](https://csrc.nist.gov/pubs/sp/800/30/r1/final)
- [MITRE ATT&CK: Groups](https://attack.mitre.org/groups/)
- [CISA: Cyber Threats and Advisories](https://www.cisa.gov/topics/cyber-threats-and-advisories)
- [ENISA: Threats and Trends](https://www.enisa.europa.eu/topics/cyber-threats/threats-and-trends)
