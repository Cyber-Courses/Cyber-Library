---
title: "Freedom versus control: the central trade-off in security"
description: "How security work weighs open access and personal autonomy against the restrictions and oversight that protection imposes."
keywords:
  - freedom versus control
  - negative and positive liberty
  - security trade-offs
  - access restrictions
  - autonomy and oversight
  - control ratchet
---

# Freedom versus control

**Freedom and control are both goods, and security work spends one to buy the other.** People want systems that are open, usable, and private from the operator's gaze; the same systems need to resist misuse, which means restricting actions, watching activity, and taking decisions out of individual hands. Almost every security choice lands somewhere on the line between those two goods, and no point on the line is free of cost.

## Negative and positive liberty, applied to a network

The philosopher Isaiah Berlin, in his 1958 lecture "Two Concepts of Liberty," split freedom into two kinds. *Negative liberty* is freedom from interference: no one stops you from acting. *Positive liberty* is freedom to actually accomplish something: having the capacity and conditions to act well.

Security touches both, and the two can point in opposite directions:

- A mandatory password manager, forced updates, and blocked USB ports **reduce negative liberty**. The user is interfered with.
- The same measures can **expand positive liberty**. A network that is not constantly compromised is one people can actually use to do their work.

The hard cases are where protecting one person's positive liberty requires interfering with another's negative liberty. "More security" is rarely a clean win; it is usually a transfer.

## The dial, not the switch

Treating freedom and control as on or off hides the real work, which is choosing a setting. The same resource can sit at very different points depending on who holds it and why.

| Setting | Freedom favored | Control favored |
|---|---|---|
| Default access | Open unless restricted | Denied unless granted |
| Monitoring | Logs kept minimal | Activity recorded in detail |
| Exceptions | User can self-approve | Every request reviewed |
| Knowledge sharing | Published openly | Shared on need-to-know |

Most security techniques are a deliberate move on one of these rows, not a jump to the far end of all of them.

## Walking a campus network up the dial

Consider a university adding controls to a research network after an incident.

1. **Start.** Any member can connect any device and reach any system. Maximum freedom, minimum assurance.
2. **First turn.** Devices must be registered and patched before they join. A clear gain in safety; a small tax on autonomy.
3. **Second turn.** Access is granted per role and expires, applying [least privilege](../principles/least-privilege.md). Researchers now file requests they used to skip.
4. **Overshoot.** Full keystroke logging is added "to be safe." Protection rises slightly; trust and morale fall sharply.

Step four shows why the destination matters: past a point, each new restriction buys little safety and spends a lot of goodwill.

## The ratchet, and the tension under other names

Controls have a ratchet quality: they are easy to add after an incident and politically hard to remove, so systems drift toward control unless someone argues for the freedom side. Naming the competing values out loud is what keeps the dial honest.

This same tension reappears across the foundation under other labels. It is [security versus privacy](security-versus-privacy.md) when the control in question is surveillance, and it is [authorization and consent](authorization-and-consent.md) when the question is who may draw the line and whether the governed agreed to it. It also shapes [dual-use knowledge](dual-use-knowledge.md), where restricting information to prevent harm also restricts the defenders who need it.

## References

- [Isaiah Berlin, *Liberty* (including "Two Concepts of Liberty"), Oxford University Press](https://global.oup.com/academic/product/liberty-9780199249893)
- [NIST SP 800-53 Rev. 5, Security and Privacy Controls (Access Control family)](https://csrc.nist.gov/pubs/sp/800/53/r5/upd1/final)
- [Saltzer and Schroeder, The Protection of Information in Computer Systems (1975)](https://www.cs.virginia.edu/~evans/cs551/saltzer/)
- [NIST SP 800-37 Rev. 2, Risk Management Framework](https://csrc.nist.gov/pubs/sp/800/37/r2/final)
- [EFF, Surveillance Self-Defense](https://ssd.eff.org/)
