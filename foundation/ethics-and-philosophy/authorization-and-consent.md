---
title: "Authorization and consent: the line between testing and intrusion"
description: "Explicit permission is what separates legitimate security testing from unlawful intrusion, defining the scope, limits, and legitimacy of every action a practitioner takes."
keywords:
  - authorization
  - consent
  - scope-of-engagement
  - rules-of-engagement
  - responsible-testing
  - security-ethics
---

# Authorization and consent

**Authorization is the explicit permission that turns an otherwise hostile act into a legitimate one.** The same technique, such as scanning a network, cracking a password, or exploiting a flaw, can be professional security work or an offense, and the difference is rarely in the technique itself. It is in whether the owner of the system agreed, in advance and in writing, to let someone act against it. Whether an unauthorized act is actually a crime depends on the conduct and the applicable law (a light scan of public services is treated very differently from cracking a password), but the ethical requirement to obtain permission does not change. Consent is the ethical and legal ground every practitioner stands on.

## Same scan, two different acts

Picture two people running the identical port scan and password-spraying attempt against the same company on the same night. Nothing in the network traffic tells them apart. What differs is everything around the keystrokes.

| | Tester with authorization | Person without it |
|---|---|---|
| Legal standing | Lawful work under a signed agreement | A likely offense under computer-misuse laws |
| Liability | Governed by the agreement and applicable law, not erased by it | Personally exposed for every action taken |
| Record | Owner knows what was agreed, by whom, and when | Owner learns about it from alerts, if at all |
| Owner's autonomy | Respected: the owner chose to be tested | Overridden: someone decided for them |

That last row is the ethical core. People and organizations have a right to decide who may probe their systems and their data, and acting without that decision takes the choice away from them, however good the intentions. It is also why [dual-use knowledge](dual-use-knowledge.md) is governed by permission rather than by skill: defender and attacker share the same capabilities.

## Who has the standing to say yes

A signature is only as good as the authority behind it. Consent from the wrong person can leave a tester exactly as exposed as no consent at all.

- **The manager without standing.** A team lead may approve testing of an application that actually runs on shared, regulated, or group-wide systems they do not control. Their sign-off does not reach those systems.
- **The landlord behind the tenant.** A tested application often runs on cloud or vendor infrastructure whose owner never signed anything. Providers set their own terms: AWS, for example, lets customers test their own resources on a list of permitted services without prior approval, but forbids testing AWS's own infrastructure and prohibits activities such as denial-of-service flooding without separate approval.
- **The people inside the data.** When testing touches real customer or employee records, the system owner's consent does not automatically cover the individuals whose data is exposed. This is where authorization meets [security versus privacy](security-versus-privacy.md).

Who is entitled to grant access to a system is itself a question of power, explored further in [freedom versus control](freedom-versus-control.md).

## What a rules-of-engagement document pins down

Authorization is captured before any work begins, usually in a scope document or rules-of-engagement agreement. A good one answers four questions in writing:

1. **Who** granted permission, and did they actually have the authority to grant it?
2. **What** systems, addresses, and accounts are in scope, and which are explicitly off-limits?
3. **When** may testing happen, and within what time window?
4. **How far** may a tester go: proof of access only, or active exploitation and data retrieval?

Testing guidance such as NIST SP 800-115 treats this planning stage as part of the assessment itself, not paperwork before it. Permission that is not documented is, for practical purposes, permission that does not exist: "everyone knows we test this" is not consent.

## Bounty programs and safe harbor: consent published to everyone

A penetration test grants consent to one named team. Bug bounty programs and coordinated vulnerability disclosure policies do something different: they publish consent to anyone who follows the rules. A safe-harbor policy tells researchers what they may test, which techniques are off the table, how to report, and what the organization promises in return, typically not to pursue legal action against good-faith research that stays within those terms.

The terms still bind. A researcher who exfiltrates more data than needed to demonstrate a flaw, or who tests an asset not listed in the policy, has stepped outside the published consent just as surely as a contracted tester who ignores their scope.

## Three moments when the boundary moves

The clean line blurs during real work, and this is where judgment matters most.

**The pivot.** A tester exploits an in-scope web server and finds credentials that open a database host never mentioned in the agreement. The find may be valuable, but the permission ends at the boundary. The right move is to record the path, stop, and ask the client whether to extend scope.

**The surprise neighbor.** An in-scope IP range turns out to include addresses now leased to a different company. Scope on paper does not match reality, and those hosts belong to someone who never consented.

**The verbal yes.** A client contact says on a call, "Go ahead and try the VPN too." Until that is confirmed in writing by someone with authority, it is implied consent, not authorization.

In every case the ethical default is the same: when authorization is unclear, stop and ask, not proceed and explain later. Even fully authorized work leaves the practitioner answerable for its consequences, which is the subject of [moral responsibility](moral-responsibility.md).

## References

- [OWASP Web Security Testing Guide](https://owasp.org/www-project-web-security-testing-guide/)
- [NIST SP 800-115, Technical Guide to Information Security Testing and Assessment](https://csrc.nist.gov/pubs/sp/800/115/final)
- [CISA: Coordinated Vulnerability Disclosure Process](https://www.cisa.gov/coordinated-vulnerability-disclosure-process)
- [EFF: Coders' Rights Project Vulnerability Reporting FAQ](https://www.eff.org/issues/coders/vulnerability-reporting-faq)
- [AWS: Penetration Testing Policy](https://aws.amazon.com/security/penetration-testing/)
