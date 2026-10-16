---
title: "Dual-use knowledge: the same skill that defends can harm"
description: "Dual-use knowledge is information or technique that can defend or harm depending on who holds it and why they use it."
keywords:
  - dual-use knowledge
  - dual-use technology
  - coordinated vulnerability disclosure
  - security through obscurity
  - offensive security
  - export controls
---

# Dual-use knowledge

**Dual-use knowledge is information or skill that protects and endangers with equal ease, so its moral weight sits not in the knowledge but in the hand that holds it.** Learning how a lock is picked teaches you both to defeat it and to choose a better one. In security almost every capability is built this way: the understanding that lets a defender find a flaw is the same understanding that lets an attacker reach it.

## The export-control origin of a loaded term

The phrase "dual-use" did not start with computers. It comes from arms control, where a dual-use good is a technology, such as a centrifuge, a chemical precursor, or a strain of pathogen, with both civilian and military uses. The Wassenaar Arrangement, a multilateral export-control regime founded in 1996, exists to track such goods as they cross borders. In 2013 its control list was expanded to cover the systems and tools used to build, command, and deliver "intrusion software," an attempt to treat such surveillance and intrusion toolkits like weapons, though the exploit payloads themselves fell outside the control. The reaction from researchers was immediate: the same code a government might buy for surveillance is what a defender uses to test and patch, so a rule written to stop one blocks the other. The episode showed how hard the category is to regulate when the item is knowledge rather than hardware.

## The same technique in two hands

No column below holds a different skill. Port scanning is port scanning. What changes is who runs it, against what, and with whose permission.

| Technique | In a defender's hands | In an attacker's hands |
| --- | --- | --- |
| Vulnerability research | Finds flaws so they can be fixed | Finds flaws so they can be exploited |
| Exploit code | Proves a risk is real and seeds detections | Turns a flaw into working intrusion |
| Reverse engineering | Extracts malware indicators | Copies a rival's methods to reuse them |
| Credential testing | Measures how weak passwords are | Steals account access |

Because the skill never shifts, the whole question collapses onto permission and purpose, which is why [authorization and consent](authorization-and-consent.md) carries so much weight in this field.

## Walking a vulnerability to the public

Consider a researcher who finds a serious flaw in widely used software. The finding is now dual-use in the sharpest sense, and what happens next decides who it helps.

1. **Hoard it.** Kept private or sold, the flaw serves whoever holds it and no one else, and the public stays exposed without knowing.
2. **Coordinate disclosure.** The researcher tells the vendor first, agrees on a timeline, and waits for a fix before details go public. Defenders gain; attackers are delayed.
3. **Publish in full.** Releasing everything at once arms defenders and attackers in the same instant, on the theory that public pressure forces a faster repair.

There is no path that helps only the good actors. That absence is the defining problem of the field.

## Why secrecy is not a safe default

The tempting response is simply to lock the knowledge away. History cuts against it. Keeping methods secret, the posture known as security through obscurity, tends to shelter attackers who will rediscover the flaw anyway while leaving honest defenders blind. Open knowledge is uncomfortable but usually safer, a judgment that runs straight into [freedom versus control](freedom-versus-control.md) and the broader weighing in [security versus privacy](security-versus-privacy.md). Deciding where that line falls, and answering for the result, is a matter of [moral responsibility](moral-responsibility.md), not a rule anyone can hand you.

## References

- [The Wassenaar Arrangement](https://www.wassenaar.org/)
- [ISO/IEC 29147:2018, Vulnerability disclosure](https://www.iso.org/standard/72311.html)
- [CISA Coordinated Vulnerability Disclosure Process](https://www.cisa.gov/coordinated-vulnerability-disclosure-process)
- [EFF Coders' Rights Project](https://www.eff.org/issues/coders-rights)
