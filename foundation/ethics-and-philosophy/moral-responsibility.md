---
title: "Moral responsibility: the duty that comes with power over systems"
description: "Moral responsibility is the duty practitioners owe to users, organizations, and the public when they hold power over systems that others cannot see, inspect, or consent to."
keywords:
  - moral responsibility
  - professional ethics
  - duty of care
  - public good
  - acm code of ethics
  - security ethics
---

# Moral responsibility

**Moral responsibility is the duty a practitioner carries because their decisions reach people who never agreed to them and often never learn of them.** A security engineer who weakens a default, a researcher who publishes an exploit, or an administrator who delays a patch is acting on behalf of users, an employer, and a wider public that is not in the room. Holding power over a system is not neutral. It creates obligations that outlast any single task, and it makes the practitioner answerable for consequences they set in motion even when no rule was broken.

## The asymmetry that creates the duty

Ordinary transactions assume rough equality: both sides can inspect what they are agreeing to. Security work breaks that assumption. The people affected by a design choice usually cannot read the code, cannot see the logs, and cannot judge whether a risk was handled well. They are dependent on someone they will never meet.

That one-sided power is the reason competence alone is not enough. Where a user cannot protect themselves, the person who can has a duty to act in their interest, not merely within their rights. Having permission to act, the subject of [authorization and consent](authorization-and-consent.md), settles whether an act is allowed; it does not settle whether it was right.

## Answering before and after the fact

Philosophers split responsibility into two directions, and both bind the practitioner at once.

| Direction | The question it asks | What it looks like in practice |
|---|---|---|
| Forward-looking | What am I obligated to do or prevent? | Choosing a safe default, disclosing a flaw, refusing to ship a known weakness |
| Backward-looking | Who must answer for what already happened? | Owning a breach, explaining a decision, accepting the fallout of a shortcut |

The forward-looking duty is the heavier one in security, because most harm is preventable before it occurs. A practitioner cannot wait for an incident to discover they were responsible all along.

## What the ACM code places first

Professional codes turn this abstract duty into a ranked order. The ACM Code of Ethics opens by telling computing professionals to "contribute to society and to human well-being" and to "avoid harm," and it states that the public good is the central concern of all professional computing work. The IEEE and (ISC)2 codes echo the same priority: when the interest of an employer, a client, and the public collide, the public comes first.

This is where intent, consequence, and competing values meet. Good intentions do not discharge the duty if the foreseeable result is harm, and a lawful instruction from an employer does not override the obligation to the people downstream.

## A flaw only you know about

Consider how the duty bites in a single, common moment. You find a serious vulnerability in a product millions of people rely on.

1. **The capability is neutral.** The same finding can be quietly weaponized or responsibly reported. The knowledge is [dual-use knowledge](dual-use-knowledge.md); what you owe is decided by what you do next.
2. **The forward-looking duty activates.** You are now one of the few who can prevent harm, which means silence is itself a choice with consequences.
3. **Answerability follows the choice.** Whether you coordinate disclosure, sell the flaw, or post it publicly, you will have to answer for who got hurt and who got protected.

No external rule forces step two or three. The pull comes from the position you are in, not from a policy.

## Where responsibility becomes accountability

Moral responsibility is the duty itself; [accountability](../principles/accountability.md) is the machinery, such as logs, ownership, and review, that makes it possible to trace a decision back to a person and have them answer for it. Responsibility is also what remains when the easy answers run out: when protecting one group exposes another, the practitioner cannot escape the weighing, a tension explored in [security versus privacy](security-versus-privacy.md). The duty does not disappear because a choice is hard; it is precisely the hard choices it exists to govern.

## References

- [ACM Code of Ethics and Professional Conduct](https://www.acm.org/code-of-ethics)
- [IEEE Code of Ethics](https://www.ieee.org/about/corporate/governance/p7-8.html)
- [(ISC)2 Code of Ethics](https://www.isc2.org/ethics)
- [Stanford Encyclopedia of Philosophy: Moral Responsibility](https://plato.stanford.edu/entries/moral-responsibility/)
