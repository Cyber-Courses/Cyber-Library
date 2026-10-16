---
title: "Authorization and consent: permission as the line between testing and intrusion"
description: "How explicit permission separates legitimate security testing from intrusion, and who may grant it, what it must cover, and when it runs out."
keywords:
  - authorization
  - consent
  - rules of engagement
  - scope of testing
  - safe harbor
  - security ethics
---

# Authorization and consent

**Authorization is the explicit permission that turns an otherwise hostile act into legitimate work.** A port scan, a cracked password, or an exploited flaw can be a paid assignment or an offense, and nothing in the packets tells the two apart. The difference sits entirely outside the keyboard: did the owner of the system agree, in advance, to be acted against? Consent is the ground a practitioner stands on, and without it even flawless technique is trespass.

## Permission, not technique, is the dividing line

Imagine two people running the identical scan against the same company on the same night. The traffic is indistinguishable; the standing behind it is not.

| | Acting with authorization | Acting without it |
|---|---|---|
| Legal footing | Lawful work under a signed agreement | A likely offense under computer-misuse law |
| The owner's choice | Respected; they chose to be tested | Overridden; someone decided for them |
| Visibility | Owner knows who, what, and when | Owner learns from alerts, if at all |

The middle row is the ethical heart of it. People and organizations have a right to decide who may probe their systems, and acting without that decision strips the choice away no matter how good the intent. It is also why [dual-use knowledge](dual-use-knowledge.md) is governed by permission and not by skill: the defender and the intruder command the same techniques.

## Whose "yes" actually counts

A signature is only as strong as the authority behind it, and consent from the wrong party leaves a tester as exposed as no consent at all.

- **A manager without reach.** A team lead may approve testing of an application that in fact runs on shared or regulated systems they do not control. Their sign-off does not extend to those systems.
- **The infrastructure owner.** Applications often sit on cloud or vendor platforms whose operator signed nothing. Cloud providers publish their own testing rules, permitting customers to test their own resources on named services while forbidding attacks on the provider's own infrastructure.
- **The people inside the data.** Testing that touches real customer records reaches individuals the system owner cannot consent for, which is exactly where authorization meets [security versus privacy](security-versus-privacy.md).

Who is entitled to grant access is itself a question of power, taken up in [freedom versus control](freedom-versus-control.md).

## What the rules of engagement must pin down

Permission is captured before any work starts, usually as a scope document or rules-of-engagement agreement. A sound one answers four questions in writing:

1. **Who** granted permission, and did they hold the authority to grant it?
2. **What** systems and accounts are in scope, and which are explicitly off-limits?
3. **When** may testing run, and inside what window?
4. **How far** may it go: proof of access only, or active exploitation and data retrieval?

Guidance such as NIST SP 800-115 treats this planning as part of the assessment, not paperwork before it. Undocumented permission is, in practice, no permission: "everyone knows we test this" is not consent.

Bug bounty and coordinated disclosure programs extend the same idea to strangers. A safe-harbor policy publishes consent to anyone who follows its terms, promising not to pursue good-faith research that stays in bounds, yet a researcher who exfiltrates more than needed or probes an unlisted asset has stepped outside it just as plainly as a contractor ignoring scope.

## When the "yes" runs out mid-engagement

The clean line blurs during real work, and judgment matters most at the edges.

1. **The pivot.** An in-scope server yields credentials to a database host no one named. The find is tempting, but permission ends at the boundary; record the path, stop, and ask before going further.
2. **The surprise neighbor.** An in-scope address range turns out to include hosts now leased to a different company that never consented.
3. **The verbal yes.** A contact says on a call, "try the VPN too." A spoken go-ahead from someone with the authority to give it is real permission, but until it is written into the scope it cannot be proven later and may not have come from someone who could actually grant it, so the VPN stays outside the rules of engagement until they are formally amended.

The default in each case is the same: when authorization is unclear, stop and ask rather than proceed and explain later. Even fully authorized work still leaves the practitioner answerable for its results, which is the subject of [moral responsibility](moral-responsibility.md).

## References

- [NIST SP 800-115, Technical Guide to Information Security Testing and Assessment](https://csrc.nist.gov/pubs/sp/800/115/final)
- [OWASP Web Security Testing Guide](https://owasp.org/www-project-web-security-testing-guide/)
- [CISA: Coordinated Vulnerability Disclosure Process](https://www.cisa.gov/coordinated-vulnerability-disclosure-process)
- [EFF: Coders' Rights Project Vulnerability Reporting FAQ](https://www.eff.org/issues/coders/vulnerability-reporting-faq)
