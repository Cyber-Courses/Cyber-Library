---
title: "White, black, and grey hats: Classifying hackers by intent"
description: "A framework that sorts security practitioners by their intent and authorization rather than by their technical skill."
keywords:
  - white hat
  - black hat
  - grey hat
  - ethical hacking
  - hacker ethics
  - authorization
---

# White, black, and grey hats

**The hat colors** are a shorthand for sorting people who break into systems by two things that technical skill alone cannot capture: their intent and their authorization. A white hat, a black hat, and a grey hat may use the exact same tools and techniques. What separates them is whether they have permission to act, and whether they aim to protect or to harm. The metaphor borrows from formulaic Western films, where heroes wore white hats and villains wore black ones.

## Two questions that decide the color

Every classification comes down to two questions asked about the same activity:

1. **Did the system owner authorize it?** A contract, a defined scope, a published disclosure policy, or a bug bounty's terms all count as permission. Curiosity, good intentions, or the fact that a system was easy to reach do not.
2. **What was the access meant to achieve?** Finding and fixing weaknesses points one way; theft, disruption, extortion, or personal gain point the other.

Crossing the two answers gives the familiar categories:

| | Protective intent | Harmful or self-serving intent |
| --- | --- | --- |
| **Authorized** | White hat: penetration testers, security researchers, and in-house defenders working under a contract, a scope, or a disclosure policy | Rare in theory, but real: an insider or contractor who abuses legitimate access |
| **Unauthorized** | Grey hat: probes a system without permission, then reports the flaw rather than exploiting it | Black hat: intrusion for gain, disruption, or other harm, which is what most people mean by "cybercrime" |

The lesson the table makes visible is that authorization, not capability, is the dividing line. The same scan that is a paid service in one context is a crime in another, which is why the [authorization and consent](../ethics-and-philosophy/authorization-and-consent.md) question matters more than any tool. It is also a direct illustration of [dual-use knowledge](../ethics-and-philosophy/dual-use-knowledge.md): the skill itself carries no color.

## From "hacker" to hats: how the labels arose

The language of hats grew out of the hacker community's long struggle over what the word "hacker" should mean. In the early decades of computing, a hacker was simply a skilled and curious programmer, the identity traced in [Origins](origins.md) and celebrated in the [Hacker Manifesto](hacker-manifesto.md). As break-ins and computer crime entered the public eye during the 1980s and 1990s, the press increasingly used "hacker" to mean "criminal," and many practitioners wanted a way to separate honest security work from malicious intrusion.

The color labels offered that separation, borrowing the costume code of Westerns. Their meanings have drifted, though. The Jargon File records "gray hat" as an occasional term for people with cracker skills operating within the law, for example in security evaluations, which is closer to today's white hat. The modern sense of grey hat, someone who acts without permission but means well, came later. By the time commercial penetration testing and bug bounty programs matured in the 2000s, "white hat" had become a recognized professional identity rather than just slang.

## Cases that refuse to fit one color

The two-question model is a simplification, and the hard cases show where it strains:

- **The unsolicited bug report.** A researcher notices a flaw in a public website, confirms it without permission, and emails the owner. The intent is protective, but the access was not authorized, so the act sits squarely in grey territory and its legal outcome depends on the jurisdiction and the owner's reaction.
- **The tester who drifts out of scope.** A hired penetration tester follows a promising lead onto a system the contract did not cover. The person is a white hat by profession, yet that specific access was unauthorized.
- **"Research" used as leverage.** Someone finds a vulnerability and asks for payment in exchange for keeping quiet. Calling it research does not change the color: the U.S. Department of Justice explicitly cites discovering vulnerabilities in order to extort their owners as an example of activity that is not in good faith.
- **The authorized insider who causes harm.** Permission alone does not make an act white hat. An employee who uses legitimate access to steal data has authorization but harmful intent.

These cases are why the community treats the colors as a vocabulary for discussion rather than a legal verdict. The norms that give them weight are explored further in [Subcultures and values](subcultures-and-values.md).

## Paths that turn grey into white

Much of modern security practice exists to give skilled, curious people a lawful route, the same drive described in [Curiosity and innovation](curiosity-and-innovation.md):

- **Coordinated vulnerability disclosure.** Programs such as the process run by CISA give researchers a defined channel to report flaws and a predictable sequence for fixes and publication.
- **Bug bounties and safe harbor terms.** By publishing a scope and promising not to pursue good-faith reporters, an organization converts would-be grey hat probing into authorized work.
- **Prosecution policy.** In May 2022 the U.S. Department of Justice announced that good-faith security research should not be charged under the Computer Fraud and Abuse Act, defining it as access solely for testing, investigating, or correcting a security flaw in a way designed to avoid harm.
- **Professional titles.** "Ethical hacker" and "security researcher" are now mainstream job titles built directly on the white hat idea, and NIST describes penetration testing as testing that mimics real-world attacks under defined conditions.

Many legal disputes still turn on whether access was authorized, which is exactly the gap the grey hat label names. Each of these mechanisms narrows that gap by making permission explicit before anyone starts probing.

## References

- [EFF: Coders' Rights Project](https://www.eff.org/issues/coders)
- [NIST Glossary: Penetration Testing](https://csrc.nist.gov/glossary/term/penetration_testing)
- [CISA: Coordinated Vulnerability Disclosure Process](https://www.cisa.gov/coordinated-vulnerability-disclosure-process)
- [U.S. Department of Justice: New Policy for Charging Cases under the Computer Fraud and Abuse Act (2022)](https://www.justice.gov/opa/pr/department-justice-announces-new-policy-charging-cases-under-computer-fraud-and-abuse-act)
- [The Jargon File: entry for "black hat"](http://www.catb.org/jargon/html/B/black-hat.html)
- [The Jargon File: entry for "hacker"](http://www.catb.org/jargon/html/H/hacker.html)
