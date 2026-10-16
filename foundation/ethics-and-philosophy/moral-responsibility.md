---
title: "Moral responsibility: the duty that comes with power over systems"
description: "The obligation security practitioners owe to users, organizations, and the public because their work gives them real power over systems that affect people's lives."
keywords:
  - moral responsibility
  - security ethics
  - professional accountability
  - duty of care
  - responsible practice
  - ethics of technology
---

# Moral responsibility

**Moral responsibility** is the duty a practitioner carries because their work gives them power over systems that other people depend on. Someone who can read a database, patch a server, or decide when to disclose a flaw is making choices that reach far beyond their own screen. Those choices touch users who never consented to the risk, organizations that trusted the practitioner with access, and a public that lives with the consequences. The more capability a person holds, the heavier that duty becomes.

## Control and knowledge: the two conditions

Philosophers since Aristotle have tied responsibility to two conditions: a person must have some **control** over what they do, and they must **know**, or be able to know, what they are doing and what it may cause. Security work satisfies both conditions unusually strongly, which is why the duty weighs more here than in most jobs.

| Condition | What it asks | Why security practitioners meet it |
| --- | --- | --- |
| Control | Could the person have acted otherwise? | Privileged access means one administrator can expose or protect millions of records, and a researcher decides what happens to a weakness they found. |
| Knowledge | Did the person understand the likely outcome? | Expertise removes the excuse of ignorance: a professional who understands the risk is accountable in a way a layperson is not. |

One further feature sharpens the duty: **harm is often invisible to the actor.** The person affected by a leaked record or a careless deletion is rarely in the room when the decision is made, so the practitioner has to keep them in mind deliberately. Being granted the keys to a system is a form of trust, not just a technical fact.

## Four habits that carry the duty

Moral responsibility rarely shows up as a grand gesture. It lives in ordinary judgment calls, and each habit below has a recognizable way of failing.

1. **Duty of care.** Handle data and access with the caution the owner would expect, even when no one is watching. The failure looks like copying a production database to a laptop "just to test something."
2. **Honest disclosure.** Report flaws, incidents, and mistakes promptly, including your own. The failure looks like quietly fixing a misconfiguration you caused and never mentioning the exposure window.
3. **Proportionate action.** Use only the access and force a task requires, and stop when the job is done. The failure looks like a penetration tester pivoting into systems outside the agreed scope because they could. Whether you were permitted to act at all is the subject of [authorization and consent](authorization-and-consent.md).
4. **Standing by consequences.** Own the outcome of a decision rather than hiding behind a policy or an order. The failure looks like "I only ran the script I was given."

These habits map onto professional codes. The (ISC)2 Code of Ethics opens with the canon to "protect society, the common good, necessary public trust and confidence, and the infrastructure," and the ACM and IEEE codes likewise ask members to put the public interest above convenience or employer pressure.

## Many hands, no one accountable

Modern systems are built and run by teams, vendors, and contractors, and responsibility can be spread so thin that no one feels it. This is known as the **problem of many hands**: when many people contribute to an outcome, it becomes hard to say who was responsible for what. Helen Nissenbaum identified it in 1996 as one of the main barriers to accountability in computing.

The classic case is the Therac-25, a radiation therapy machine whose software flaws contributed to a series of massive overdoses in the mid-1980s. Developers, testers, the manufacturer, and operators each held a piece of the failure, and no single person's choice explains it alone. In security, the same pattern appears when a known weakness sits in a backlog for months because each team assumes another one owns it. The practical answer is to name an owner for every risk and to treat "not my component" as a question to resolve, not a conclusion.

## When the employer wants silence: a disclosure scenario

Consider an engineer who finds that a customer-facing service leaks other customers' account details. Management asks her to keep it quiet until the next quarterly release, three months away.

- **The pull of loyalty.** She owes her employer diligent, competent service, and a premature public announcement could hand attackers a map before a fix exists.
- **The pull of the public.** Every day of silence leaves users exposed to a risk they do not know about and cannot defend against. Her codes of ethics put their interest first.
- **The cost of each path.** Acting on a known risk can itself cause harm, and so can waiting. Inaction is a choice too, and it carries moral weight.

A defensible path usually lies between silence and a public leak: push internally for an urgent fix, document the risk and who accepted it, and if that fails, use a structured channel such as the coordinated disclosure process that CISA runs, which exists precisely to fix flaws before details go public. The same tension between protecting people and exposing them appears in [security versus privacy](security-versus-privacy.md), and the question of how much power over others a practitioner should hold in the first place is taken up in [freedom versus control](freedom-versus-control.md).

## Orders do not transfer responsibility

Following instructions does not move the moral weight onto whoever gave them. A lawful order can still be the wrong thing to do, and "I was told to" explains a choice without excusing it. This matters most for practitioners whose skills cut both ways: the expertise that hardens a system can also be turned against it, as [dual-use knowledge](dual-use-knowledge.md) explores, so the person holding that expertise cannot hand the decision about its use entirely to someone else.

There is no formula that resolves these conflicts cleanly. Moral responsibility is the willingness to weigh the competing claims honestly and to answer for the choice.

## References

- [(ISC)2 Code of Ethics](https://www.isc2.org/ethics)
- [ACM Code of Ethics and Professional Conduct](https://www.acm.org/code-of-ethics)
- [IEEE Code of Ethics](https://www.ieee.org/about/corporate/governance/p7-8.html)
- [Stanford Encyclopedia of Philosophy: Moral Responsibility](https://plato.stanford.edu/entries/moral-responsibility/)
- [Stanford Encyclopedia of Philosophy: Computing and Moral Responsibility](https://plato.stanford.edu/entries/computing-responsibility/)
- [CISA: Coordinated Vulnerability Disclosure Process](https://www.cisa.gov/coordinated-vulnerability-disclosure-process)
