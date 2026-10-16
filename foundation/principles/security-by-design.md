---
title: "Security by Design: Building Protection in From the Start"
description: "The principle that security should be planned into a system's architecture from the beginning rather than added after it is built."
keywords:
  - security by design
  - secure by design
  - threat modeling
  - secure development lifecycle
  - security architecture
  - default security
---

# Security by design

**Security by design** means treating protection as a core requirement of a system from its earliest planning stages, rather than as a feature bolted on once the system already works. When security is considered alongside functionality, performance, and cost, the resulting architecture tends to be simpler, cheaper to defend, and far less brittle than one patched after release. The idea is closely tied to the related goal of being secure by default, where a system ships in its safest configuration and leaves users protected without extra effort.

## The 1975 design principles behind the idea

The notion is older than the modern slogan. In 1975, Jerome Saltzer and Michael Schroeder published "The Protection of Information in Computer Systems," which set out design principles for protection mechanisms. Several of them read today as the core of security by design:

| Saltzer and Schroeder principle | What it asks of the designer |
| --- | --- |
| Economy of mechanism | Keep the design small and simple so it can actually be reviewed. |
| Fail-safe defaults | Base access on explicit permission; the default answer is "no." |
| Complete mediation | Check every access to every object, not just the first one. |
| Open design | Do not rely on the secrecy of the design itself for protection. |
| Least privilege | Give each program and user only the rights the task needs. |
| Psychological acceptability | Make the secure path easy, or people will route around it. |

None of these can be added cleanly to a finished system. A sprawling codebase cannot be made economical after the fact, and an access model that allows by default cannot be flipped to deny by default without breaking things that came to depend on it.

## Decisions that are cheap early and costly late

Fixing a flaw grows more expensive and disruptive the later it is found. A weakness caught in a design review can be redrawn on a whiteboard; the same weakness discovered in production may require emergency patches, downtime, and incident response. The choices below are the ones where that gap is widest:

- **Trust boundaries.** Deciding which components trust each other, and where input must be validated, shapes every interface that follows.
- **Data flows.** Where sensitive data travels and where it is stored determines what must be encrypted, logged, and protected.
- **Authentication and access model.** How identities are established and permissions granted is woven through the whole system and is painful to reverse.
- **Attack surface.** Every feature, port, and service that ships becomes something an attacker can target and something the team must maintain.

Many serious vulnerabilities come from architectural choices like these, not from coding slips, and no amount of later testing can fully undo a weak foundation.

## Walking a feature through a secure design

Consider a team adding a "share document by link" feature to a web application. Applying security by design looks roughly like this:

1. **Model the threats before writing code.** The team asks what it is protecting (private documents), who might attack it (anyone who obtains or guesses a link), and how (enumerating links, reusing old ones, forwarding them).
2. **Choose the safe default.** Links are created private to named recipients; public links are an option the owner must deliberately turn on.
3. **Shrink the surface.** Links are long and random, expire after a set time, and grant read access only, so the feature exposes nothing it does not need.
4. **Fail closed.** If the permission service is unreachable, the link returns an error instead of serving the document.
5. **Keep checking through the lifecycle.** The design is reviewed, the code is analyzed and tested against the threat model, and the threats are revisited when the feature changes.

Step 5 is what a secure development lifecycle formalizes: security tasks such as design review, code analysis, and testing are built into each phase of development rather than saved for the end.

## Secure by default and the shift to vendor responsibility

Being secure by default is the part of the principle users feel directly. Systems start in a locked-down state, with risky features off until a user deliberately enables them. The opposite is a system that ships with sample passwords, open permissions, or verbose error messages and stays exposed until someone manually hardens it.

In April 2023, the US Cybersecurity and Infrastructure Security Agency (CISA) and partner agencies from several countries published joint guidance urging software manufacturers to adopt secure by design and secure by default practices. Its three principles push responsibility away from customers and toward vendors:

- **Take ownership of customer security outcomes**, rather than leaving hardening to the people who buy the product.
- **Embrace radical transparency and accountability**, for example by being open about vulnerabilities and how they are handled.
- **Lead from the top**, so that security is a business priority set by executives, not only a task for engineers.

## Signs security was bolted on afterward

A system built without security by design tends to show recognizable symptoms:

- **Security is a final checkpoint.** A review just before launch is the only control, when the architecture is already fixed and hard to change.
- **Trust is assumed, not designed.** Internal networks, client software, or input from other components are treated as safe, which leaves no defense when that assumption fails.
- **Hardening is the customer's job.** The product only becomes safe after someone follows a long configuration guide.
- **No one can explain the whole system.** Features and integrations have piled up until no one can reason about how it behaves under attack.

Security by design is also the stage where other principles get decided. It plans the layered controls that [defense in depth](defense-in-depth.md) relies on, settles who gets what access under [least privilege](least-privilege.md), and makes the reduction of data and attack surface described in [minimization](minimization.md) a deliberate choice. [Zero trust](zero-trust.md) is a modern architecture that applies the same thinking by removing implicit trust from the start.

## References

- [Saltzer and Schroeder, The Protection of Information in Computer Systems (1975)](https://www.cs.virginia.edu/~evans/cs551/saltzer/)
- [CISA: Secure by Design](https://www.cisa.gov/securebydesign)
- [CISA: Principles and Approaches for Secure by Design Software](https://www.cisa.gov/resources-tools/resources/secure-by-design)
- [NIST SP 800-160 Vol. 1, Engineering Trustworthy Secure Systems](https://csrc.nist.gov/pubs/sp/800/160/v1/r1/final)
- [OWASP: Secure Product Design Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Secure_Product_Design_Cheat_Sheet.html)
- [OWASP: Threat Modeling Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Threat_Modeling_Cheat_Sheet.html)
