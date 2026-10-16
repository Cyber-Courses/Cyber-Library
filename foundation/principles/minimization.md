---
title: "Minimization: Keeping Only What Is Required"
description: "The security principle of reducing data, access, and attack surface so that a system holds and exposes only what it genuinely needs."
keywords:
  - minimization
  - data minimization
  - attack surface reduction
  - least functionality
  - privacy by design
  - security principles
---

# Minimization

**Minimization** is the practice of keeping a system as small as it can be and still do its job: less data stored, less access granted, and fewer exposed features, interfaces, and components. The reasoning is direct. Anything a system does not hold cannot be stolen, anything it does not expose cannot be attacked, and anything it does not permit cannot be abused. Minimization shows up across security and privacy work, from data protection regulation that requires collecting only what is necessary to hardening guides that strip unused services from a server.

## Three things to shrink

Minimization is one idea applied to three different kinds of "stuff." Each has its own guiding question and its own typical controls.

| What you shrink | The question to ask | Typical measures |
| --- | --- | --- |
| **Data** | Does this purpose actually need this field, and for how long? | Collect only required fields, set retention periods, delete or archive when the purpose is served |
| **Functionality** | Does this service, protocol, account, or package need to be running? | Uninstall unused software, disable sample code and debug features, remove default accounts |
| **Exposure** | Who needs to reach this interface, and from where? | Close unused ports, publish the narrowest set of APIs and endpoints, keep internal systems off the public network |

The payoff differs slightly for each. Less data means a breach exposes less: a database that keeps only current, necessary records leaks far less than one holding years of unused personal data. Less functionality and less exposure mean a smaller attack surface, because every open port, installed package, API endpoint, and enabled feature is a potential way in. All three lower cost and complexity, since there is less to patch, monitor, back up, and reason about.

## Asking "do we need this?" across the data lifecycle

Data minimization is not a single decision made at collection time. The same question returns at every stage a piece of data passes through:

1. **Collection.** Gather only the fields the process truly needs. Resist the "just in case it is useful later" field; it becomes a liability long before it becomes useful.
2. **Use and access.** Give each person and service access to the narrowest slice of data their task requires, so that even granted access reaches as little as possible.
3. **Storage.** Prefer forms that hold less: store a derived value or a truncated identifier instead of the full original when that is enough.
4. **Sharing and copying.** Every export, analytics feed, or third-party integration is a new copy that must be minimized in its own right.
5. **Deletion.** Decide at design time when the data will be removed and build the mechanism that does it. Data collected with no deletion plan simply accumulates.

## Where the extra copies and features hide

Systems rarely grow bloated through one big decision. They grow through habits and side channels that escape the review applied to the main design:

- **Collect-everything defaults:** teams gather data by default, assuming more is always better, and build a store they never use but must still protect.
- **Shadow copies:** backups, logs, exports, and analytics pipelines quietly duplicate sensitive data outside the systems that were carefully minimized.
- **Forgotten retention:** stores keep growing because no job ever deletes anything.
- **Feature sprawl in production:** optional modules, debug endpoints, and sample code ship enabled because nobody turned them off.

A minimized primary database offers little protection if a nightly log file contains the same records in full. Reviewing these side channels is often where the real reduction happens.

## From regulation to hardening baselines

Minimization is written into both privacy law and security control catalogs, which is one reason it appears in such different contexts:

- **Privacy law.** The EU General Data Protection Regulation (GDPR) requires in Article 5 that personal data be "adequate, relevant and limited to what is necessary" for its purposes (data minimization) and kept no longer than necessary (storage limitation). Similar expectations exist in many other jurisdictions and form a core part of privacy by design.
- **Security controls.** NIST SP 800-53 includes control CM-7, Least Functionality, which directs organizations to configure systems to provide only essential capabilities and to prohibit or restrict unneeded functions, ports, protocols, and services.
- **Attack surface analysis.** Guidance such as the OWASP Attack Surface Analysis Cheat Sheet treats mapping and then reducing entry points as a routine part of application security.

## Minimization alongside the other principles

Minimization rarely works alone. [Least privilege](least-privilege.md) applies the same reasoning to permissions that minimization applies to data and attack surface, and the two are usually designed together. In a [defense in depth](defense-in-depth.md) strategy, a smaller system gives each layer less to protect and fewer gaps to cover. Minimization is also one of the earliest choices made under [security by design](security-by-design.md), since removing a component later is far harder than never adding it. Finally, [zero trust](zero-trust.md) complements it: limiting what is exposed pairs naturally with verifying every request against the smallest necessary access. For the broader idea of what an attacker can reach, see [assets and attack surface](../concepts/assets-and-attack-surface.md).

## References

- [NIST SP 800-53 Rev. 5, Security and Privacy Controls (CM-7 Least Functionality)](https://csrc.nist.gov/pubs/sp/800/53/r5/upd1/final)
- [NIST Glossary: Attack Surface](https://csrc.nist.gov/glossary/term/attack_surface)
- [OWASP: Attack Surface Analysis Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Attack_Surface_Analysis_Cheat_Sheet.html)
- [Regulation (EU) 2016/679, General Data Protection Regulation (EUR-Lex)](https://eur-lex.europa.eu/eli/reg/2016/679/oj)
- [UK Information Commissioner's Office: Data minimisation principle](https://ico.org.uk/for-organisations/uk-gdpr-guidance-and-resources/data-protection-principles/a-guide-to-the-data-protection-principles/data-minimisation/)
