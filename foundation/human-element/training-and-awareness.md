---
title: "Training and awareness: Closing the human knowledge gap"
description: "The knowledge gap that leaves people unable to recognize threats, and the ongoing education that helps them spot and respond to attacks."
keywords:
  - security awareness
  - security training
  - phishing awareness
  - human error
  - security culture
  - social engineering
---

# Training and awareness

**Training and awareness** describe the ongoing effort to give people the knowledge and habits they need to recognize and respond to security threats. Most technical controls still depend on a human making a safe choice. When that knowledge is missing, attackers do not need to defeat the technology; they can simply ask a person to let them in. The gap between what people are expected to handle and what they have actually been taught is one of the most common contributors to security incidents.

## The split-second decisions technology hands to people

Training is easiest to design when you start from the moments where a person, not a system, decides the outcome. Phishing and other social engineering attacks aim at exactly these moments, because deceiving one person is often easier than breaking a well-defended system.

- **Click or ignore.** A link or attachment arrives that looks routine but is not.
- **Approve or deny.** A login prompt appears on a phone that the user did not trigger.
- **Share or refuse.** A caller claiming to be from IT asks for a password or a one-time code.
- **Pay or verify.** An "urgent" request asks finance to change a supplier's bank details.
- **Report or stay quiet.** Something looks wrong, and the person must decide whether it is worth raising.

Every one of these is a judgment call that no filter fully removes. What people know at that moment, and whether they feel safe acting on doubt, is what training exists to shape.

## Awareness, training and role-specific skill are not the same thing

Programs often treat "awareness" as the whole job. Guidance such as NIST SP 800-50 Rev. 1 and the Awareness and Training (AT) control family in NIST SP 800-53 distinguish between general awareness for everyone and deeper, role-based training for people whose jobs carry specific risks.

| Layer | Who it targets | What it should produce | Example |
| --- | --- | --- | --- |
| Awareness | Everyone, from day one | Recognition: "this looks wrong" | Spotting a spoofed sender or a fake login page |
| Training | Staff in a given role | A practiced, correct response | Finance verifying bank detail changes by phone through a known number |
| Role-specific skill | Builders and administrators | Safe work by default | Developers keeping secrets out of code; admins applying safe configuration baselines |

Generic content that ignores the middle and bottom rows leaves finance staff hearing nothing about invoice fraud and developers hearing nothing about secrets in code. The deeper layers connect directly to [design and development errors](design-and-development-errors.md) and [misconfiguration](misconfiguration.md), since many of those mistakes trace back to people who were never taught safe defaults.

## Five ways a program quietly fails

1. **Access before explanation.** New staff receive credentials before anyone covers password hygiene or how to report an incident.
2. **The annual checkbox.** A single yearly module is quickly forgotten and rarely reflects current attack techniques.
3. **One script for every role.** Content ignores the specific threats each team actually faces.
4. **Fear of blame.** People who expect punishment hide their mistakes, so a clicked link goes unreported until the damage spreads.
5. **No practice.** Lessons are never rehearsed, so knowledge fades long before it is needed.

A further, subtler failure is unexplained rules. A user who does not understand why a rule exists is more likely to work around it, quietly weakening the controls around them. That drift is explored in [user behavior and misuse](user-behavior-and-misuse.md).

## Keeping knowledge alive between incidents

Effective programs treat learning as a cycle rather than an event:

- **Start at onboarding, then refresh in short bursts.** Brief, regular sessions beat one long annual course.
- **Tailor examples to each team.** Use the attacks that team is likely to see, written in its own vocabulary.
- **Rehearse with simulations.** Controlled phishing exercises and tabletop drills build recognition that passive reading cannot.
- **Make reporting a one-click, no-blame habit.** Give people a simple, well-known way to raise concerns, and treat every report as a win. Agencies such as CISA and the UK NCSC publish public guidance on recognizing and reporting phishing that can seed this material.
- **Let results steer the next round.** Focus future sessions on the topics and teams where understanding is weakest.

Over time, the goal is a culture where security is a shared habit rather than a checkbox, so good decisions happen even in situations no training anticipated.

## Reports, not just clicks: measuring whether it works

Simulation click rates are the most visible metric, but on their own they can mislead: a low click rate may simply mean the test was easy. Reporting rates matter as much, because a person who recognizes a suspicious message and reports it turns into a sensor, giving defenders early warning that tools alone may miss. Tracking how many people report, how quickly, and whether reports keep rising after a simulation shows whether the culture is changing, not just whether people memorized a lesson.

Training never reaches 100 percent, which is why it pairs with architectural safeguards. Approaches like [zero trust](../principles/zero-trust.md) and [least privilege](../principles/least-privilege.md) limit how much any single uninformed decision can expose, so a trained workforce and a forgiving design reinforce each other.

## References

- [NIST SP 800-50 Rev. 1, Building a Cybersecurity and Privacy Learning Program](https://csrc.nist.gov/pubs/sp/800/50/r1/final)
- [NIST SP 800-53 Rev. 5, Security and Privacy Controls (AT, Awareness and Training)](https://csrc.nist.gov/pubs/sp/800/53/r5/upd1/final)
- [CISA: Recognize and Report Phishing](https://www.cisa.gov/secure-our-world/recognize-and-report-phishing)
- [UK NCSC: Phishing scams, how to spot and report them](https://www.ncsc.gov.uk/collection/phishing-scams)
- [SANS Security Awareness](https://www.sans.org/security-awareness-training/)
- [ENISA Threat Landscape](https://www.enisa.europa.eu/topics/cyber-threats/threats-and-trends)
