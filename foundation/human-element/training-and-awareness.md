---
title: "Training and awareness: closing the knowledge gap attackers rely on"
description: "The knowledge gap that leaves people unable to recognize threats, and the layered education that turns everyday staff into alert defenders."
keywords:
  - security awareness
  - security training
  - learning continuum
  - phishing recognition
  - human knowledge gap
  - reporting culture
---

# Training and awareness

**Training and awareness** are how an organization closes the gap between what people are expected to handle and what they have actually been taught. Most security tools still hand the final decision to a human: click or don't, approve or deny, pay or verify. When the knowledge behind that decision is missing, an attacker does not have to defeat the technology. They can simply ask a person who was never taught to say no.

## What NIST calls the learning continuum

Guidance such as NIST SP 800-50 describes learning as a continuum rather than a single event. Each stage asks for more than the last, and skipping the middle is where programs usually break.

| Stage | Audience | What success looks like |
| --- | --- | --- |
| Awareness | Everyone | Noticing that something is off: a spoofed sender, an unexpected login prompt |
| Training | People in a specific role | A practiced, correct response, such as finance verifying a bank-detail change by a known phone number |
| Education | Builders and administrators | Safe work by default, woven into how the job is done |

A program that delivers only the top row leaves finance staff hearing nothing about invoice fraud and developers hearing nothing about secrets left in code. Those deeper gaps feed directly into [design and development errors](design-and-development-errors.md) and [misconfiguration](misconfiguration.md), where the root cause is often a person who was never shown the safe default.

## Walking one phishing email through an untrained team

The cost of the gap is easiest to see by following a single message:

1. **It arrives.** An email imitates a known supplier and asks an accounts clerk to update payment details "before the next run."
2. **It is believed.** No one ever explained that urgency plus a payment change is the classic fraud pattern, so the clerk treats it as routine.
3. **It is acted on.** The details are changed. A filter never flagged the message because nothing about it was technically malicious.
4. **It is hidden.** When the mistake surfaces days later, the clerk fears blame and stays quiet, so the loss grows before anyone responds.

Awareness would have stopped the message at step two. A no-blame reporting habit would have shortened step four. Both are learned, not installed.

## The forgetting curve versus the annual module

The most common program is a single yearly course, and it fights human memory. People forget most of what they hear within days unless it is refreshed and used. The fix is not a longer course but a steadier rhythm: short refreshers, examples drawn from the threats each team actually sees, and simulated phishing or tabletop drills that force recognition to be practiced rather than merely read. Unexplained rules erode the same way; a person who does not understand why a control exists tends to work around it, a drift examined in [user behavior and misuse](user-behavior-and-misuse.md).

## Measuring recognition, not just clicks

Simulation click rates are visible but shallow: a low rate may only mean the test was easy. The stronger signal is reporting. A person who spots a suspicious message and reports it becomes a sensor, giving defenders warning that tools missed. Tracking how many people report, and how fast, shows whether the culture is actually changing.

No program reaches everyone, which is why training pairs with architecture. Designs built on [least privilege](../principles/least-privilege.md) limit how much a single uninformed decision can expose, so a forgiving system and an educated workforce cover each other's gaps.

## References

- [NIST SP 800-50 Rev. 1, Building a Cybersecurity and Privacy Learning Program](https://csrc.nist.gov/pubs/sp/800/50/r1/final)
- [NIST SP 800-53 Rev. 5, Security and Privacy Controls (AT, Awareness and Training)](https://csrc.nist.gov/pubs/sp/800/53/r5/upd1/final)
- [CISA: Recognize and Report Phishing](https://www.cisa.gov/secure-our-world/recognize-and-report-phishing)
- [UK NCSC: Phishing scams, how to spot and report them](https://www.ncsc.gov.uk/collection/phishing-scams)
- [ENISA Threat Landscape](https://www.enisa.europa.eu/topics/cyber-threats/threats-and-trends)
