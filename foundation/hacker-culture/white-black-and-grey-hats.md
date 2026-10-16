---
title: "White, black, and grey hats: intent and authorization, not skill"
description: "A guide to the color-coded labels that sort hackers by whether they have permission and what they intend, rather than by how skilled they are."
keywords:
  - white hat
  - black hat
  - grey hat
  - ethical hacking
  - authorization and intent
  - hacker classification
---

# White, black, and grey hats

**White, black, and grey hats** are labels that sort people who probe or break into computer systems by two things: whether they have permission to be there, and what they intend to do once they are. They say nothing about talent. A criminal intruder and a hired security tester can run the exact same tools; the hat describes which side of a line each one stands on.

## From Western films to security jargon

The color metaphor is older than computing. In early American Western films, the hero often wore a white hat and the villain a black one, a visual shorthand so an audience could tell them apart at a glance. Security culture borrowed the image because it captured the thing that mattered: moral stance, not ability. The labels attached themselves to the hacker identity as that identity spread, a story traced in [Origins](origins.md) and across the branches described in [Subcultures and values](subcultures-and-values.md).

## The line is authorization, not skill

Because a white hat and a black hat may use identical techniques, skill cannot separate them. Two questions do. First, authorization: were they invited, under a contract, a bug bounty scope, or explicit permission? Second, intent: did they mean to strengthen the system or to exploit it? The grey hat sits in the uncomfortable gap where one answer is missing.

| Hat | Authorization | Intent | Typical role |
|---|---|---|---|
| White | Yes, within an agreed scope | Improve the system's security | Penetration tester, bug bounty researcher |
| Black | None | Personal gain, disruption, or harm | Criminal intruder |
| Grey | None, or ambiguous | Often to expose a flaw, not to profit | Uninvited researcher who reports what they find |

The grey hat is defined by contradiction: good intent paired with no permission. That mix is why the category is debated rather than celebrated.

## One flaw, three responses

Imagine someone notices that a hospital's patient portal leaks records through a predictable web address. The same discovery branches three ways:

1. **White hat.** The finder was already engaged to test the portal, or submits through an open bug bounty program. They document the flaw, report it privately, and let the hospital fix it before any details are public.
2. **Black hat.** The finder harvests the records to sell or to extort the hospital. The act is unauthorized and the intent is harm.
3. **Grey hat.** The finder was never invited but means no harm. They confirm the flaw and email the hospital, or post a redacted warning. The intent leans ethical, yet accessing the records without permission may still break the law.

Only the first path keeps both answers clean. The third shows how good intentions do not erase the authorization problem, a tension examined in [authorization and consent](../ethics-and-philosophy/authorization-and-consent.md).

## Where the colors blur

The hats are best read as descriptions of acts, not permanent identities. A professional tester who stays strictly in scope at work might, on their own time, poke at a site uninvited and drift into grey. Terms like "red team" and "blue team" describe cooperating roles inside an authorized exercise and sit apart from this scale entirely.

The labels also carry the field's long argument over whether unauthorized curiosity is ever justified, the same defiance voiced in the [Hacker Manifesto](hacker-manifesto.md). For a learner, the useful takeaway is simple: do not judge a hacker by the sophistication of what they did, but by whether they were allowed to do it and why they did it.

## References

- [The Jargon File: cracker](http://www.catb.org/jargon/html/C/cracker.html)
- [Eric S. Raymond, How To Become A Hacker](http://www.catb.org/~esr/faqs/hacker-howto.html)
- [NIST SP 800-115, Technical Guide to Information Security Testing and Assessment](https://csrc.nist.gov/pubs/sp/800/115/final)
- [CISA Coordinated Vulnerability Disclosure Process](https://www.cisa.gov/coordinated-vulnerability-disclosure-process)
- [IETF RFC 1392: Internet Users' Glossary](https://www.rfc-editor.org/rfc/rfc1392)
