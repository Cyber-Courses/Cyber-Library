---
title: "Dual-use knowledge: the same skill defends and attacks"
description: "Information and technique in security that can defend or harm depending on who holds it and why they are using it."
keywords:
  - dual-use knowledge
  - dual-use technology
  - offensive security
  - responsible disclosure
  - security research ethics
  - knowledge and harm
---

# Dual-use knowledge

**Dual-use knowledge** is information or technique that can defend or harm depending on who holds it and why. The same understanding that lets a defender find and fix a weakness lets an attacker find and exploit it. The knowledge itself is neutral. Its effect comes from intent, context, and consequence, which is why questions of [moral responsibility](moral-responsibility.md) and [authorization and consent](authorization-and-consent.md) sit so close to this topic.

## One skill, two directions

Almost every core security skill has a mirror image. The technique does not change between the two columns; only the holder, the target, and the permission do.

| Skill | Defensive use | Offensive use |
| --- | --- | --- |
| Network mapping and port scanning | An administrator inventories exposed services | An intruder looks for an entry point |
| Exploit development | A tester proves a flaw is real so it gets fixed, and defenders write detections for it | An attacker turns the flaw into access |
| Reverse engineering malware | An analyst extracts indicators and builds signatures | A criminal studies a rival's code to reuse its tricks |
| Password cracking | An auditor measures how weak an organization's passwords are | A thief recovers credentials from a stolen database |
| Phishing techniques | A security team runs awareness simulations | A fraudster steals logins |

## Why defense cannot avoid learning attack

You cannot defend against a technique you do not understand. Threat models, red teaming, detection rules, and patches all require knowing how a system breaks, so the field depends on people building, sharing, and publishing offensive knowledge.

Openness is also how the field improves. Public research, shared tooling, and disclosed flaws raise everyone's baseline, and a defender working alone would always be a step behind attackers who share freely among themselves. The cost is that the audience cannot be chosen: a conference talk, a blog post, or a released tool reaches defenders and attackers alike.

## Levers for releasing what you know

Practitioners manage dual-use knowledge through norms and judgment rather than a single rule. The main levers, roughly from individual choice to state policy:

1. **Coordinated disclosure.** A researcher reports a flaw to the vendor and allows time for a fix before publishing details. Some teams set fixed deadlines; Google Project Zero, for example, uses a "90+30" policy: vendors get 90 days to fix a flaw, details follow 30 days after a patch ships (or at the 90-day mark if none does), and flaws already exploited in the wild get a 7-day deadline.
2. **Proof-of-concept restraint.** Demonstrating that a weakness exists without shipping a turnkey weapon for it.
3. **Tooling with guardrails.** Offensive frameworks exist openly, but their authors document legitimate use and legal boundaries.
4. **Export controls.** Some capabilities are treated as controlled goods. In 2013 the Wassenaar Arrangement, a multilateral export control regime for conventional arms and dual-use goods and technologies, added certain intrusion software to its control lists.

## Worked case: a flaw in a router nobody patches

A researcher finds a remote takeover flaw in a home router whose manufacturer has stopped supporting it. Each option carries a cost:

- **Report and wait.** The vendor may never answer, so waiting protects no one and leaves owners unaware.
- **Publish full details.** Owners and internet providers learn they must replace or isolate the device, but attackers get a recipe at the same moment.
- **Publish a warning without technical detail.** This lowers the risk of copycats, yet makes it harder for defenders to write detections or confirm exposure.
- **Sell the finding.** Selling capability is lawful in many places, but the buyer may be a security team, a government, or a broker, and the seller rarely controls where it ends up.

The case shows where reasonable people disagree. Detail helps defenders act fast and attackers act faster. Disclosure deadlines assume vendors can and will patch, which is not always true for critical infrastructure or abandoned software. And restricting the knowledge entirely can chill legitimate research without stopping determined adversaries who ignore the rules, a theme explored further in [freedom versus control](freedom-versus-control.md). The same double edge appears in [security versus privacy](security-versus-privacy.md), where one monitoring capability serves both protection and intrusion.

There is no setting that makes knowledge safe. The responsibility sits with how it is shared and used, not with pretending it could be unknown.

## References

- [CISA: Coordinated Vulnerability Disclosure Process](https://www.cisa.gov/coordinated-vulnerability-disclosure-process)
- [FIRST Guidelines and Practices for Multi-Party Vulnerability Coordination and Disclosure](https://www.first.org/global/sigs/vulnerability-coordination/multiparty/guidelines-v1.1)
- [Google Project Zero: Vulnerability Disclosure Policy](https://googleprojectzero.blogspot.com/p/vulnerability-disclosure-policy.html)
- [The Wassenaar Arrangement: Control Lists](https://www.wassenaar.org/control-lists/)
- [Wikipedia: Dual-use technology](https://en.wikipedia.org/wiki/Dual-use_technology)
