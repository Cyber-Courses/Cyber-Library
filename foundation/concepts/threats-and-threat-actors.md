---
title: "Threats and Threat Actors: Who Attacks and Why"
description: "What a threat is, the kinds of actors behind them, and the motivations and capabilities that make a threat real rather than hypothetical."
keywords:
  - threat
  - threat actor
  - threat modeling
  - adversary motivation
  - attacker capability
  - cyber threat intelligence
---

# Threats and threat actors

**A threat is any circumstance or event with the potential to harm a system**, and a threat actor is the person or group that would make it happen. Security work starts here, because you cannot defend against danger you have not named. A threat is not the weakness itself and not the damage itself; it is the possibility that sits between them, waiting on someone with a reason to act.

## A threat is not a weakness or a loss

These three ideas are easy to blur, so it helps to pull them apart:

- A **threat** is the potential for harm, such as "an attacker could encrypt our files for ransom."
- A [vulnerability](vulnerabilities-and-exploits.md) is the flaw that would let it succeed, such as an unpatched server.
- The **harm** is the realized loss, measured against the [CIA triad](cia-triad.md): data disclosed, altered, or made unavailable.

A threat with no matching weakness stays hypothetical. A weakness that no actor cares to exploit stays dormant. For a deliberate attack, risk appears only when both line up over an [asset](assets-and-attack-surface.md) worth something. Non-adversarial threats break that pattern: a power cut, a fire, a failed disk, or an accidental deletion can harm the same asset with no attacker and no exploited weakness at all, which is why risk assessments weigh accidents and the environment alongside hostile actors.

## The actors, sorted by what drives them

Who is behind a threat shapes everything about how it behaves: how patient they are, how much they will spend, and what they are willing to break. Threat intelligence groups actors less by their tools than by their motive.

| Actor | What they want | Typical capability |
| --- | --- | --- |
| Cybercriminals | Money, through fraud, theft, or ransom | Moderate to high; often buy tools and access |
| Nation-state groups | Espionage, sabotage, strategic advantage | Very high; patient, well funded, custom tooling |
| Hacktivists | Attention for a cause | Low to moderate; favor disruption and leaks |
| Malicious insiders | Grievance or profit | Variable, but start with legitimate access |
| Opportunists | Whatever is easy, including bragging rights | Low; rely on known flaws and automation |

The careless insider is a reminder that not every actor is hostile. An employee who emails a spreadsheet to the wrong address is still a source of harm your defenses must account for.

## Capability, intent, and opportunity

For a threat to be credible, three conditions have to hold at once. Analysts sometimes call this the threat triangle:

1. **Capability.** The actor has the skills, tools, or resources to carry the attack out.
2. **Intent.** The actor has a reason to target you rather than someone else.
3. **Opportunity.** A reachable weakness exists for them to use.

Defenders rarely change an adversary's capability or intent. What they can shrink is opportunity, which is why so much of security is about closing openings before anyone with a motive finds them.

## Following a ransomware crew

Watch how the pieces combine in one common pattern:

1. **Motive.** A criminal group wants payment, so any organization that cannot afford downtime is a target.
2. **Entry.** They phish an employee, exploiting the cheapest opportunity rather than the fanciest flaw.
3. **Spread.** With one foothold, they move through the network toward valuable data.
4. **Leverage.** In modern double-extortion attacks they exfiltrate the data first, breaking confidentiality, then encrypt the systems to break availability, so a victim with good backups still faces the threat of a leak.

At each step, a [control](controls-and-countermeasures.md) could have removed the opportunity: training against phishing, [least privilege](../principles/least-privilege.md) to limit the spread, and offline backups to blunt the ransom. The actor's motive never changed; the defenders simply gave it nowhere to land.

## References

- [NIST SP 800-30 Rev. 1, Guide for Conducting Risk Assessments](https://csrc.nist.gov/pubs/sp/800/30/r1/final)
- [MITRE ATT&CK Groups](https://attack.mitre.org/groups/)
- [CISA, Cyber Threats and Advisories](https://www.cisa.gov/topics/cyber-threats-and-advisories)
- [ENISA Threat Landscape](https://www.enisa.europa.eu/topics/cyber-threats/threats-and-trends)
- [Verizon Data Breach Investigations Report (DBIR)](https://www.verizon.com/business/resources/reports/dbir/)
