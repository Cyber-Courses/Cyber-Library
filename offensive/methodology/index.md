---
title: "Methodology: how offensive engagements are structured"
description: "The methodology category covers the process of an offensive engagement rather than individual techniques, split by phase: preparation, operational execution, and closure."
keywords:
  - penetration testing methodology
  - red team process
  - engagement phases
  - rules of engagement
  - reporting
---

# Methodology

The methodology category is about the process of running an offensive engagement, not the individual techniques it employs. Techniques live in the target-focused categories ([software](../software/index.md), [network](../network/index.md), [physical](../physical/index.md), [hardware](../hardware/index.md), [data](../data/index.md)); methodology is the connective tissue that decides when and how they are applied, how an operation stays within scope, and how its results become something an organization can act on.

## Why it is split this way

The subcategories follow the lifecycle of an engagement, in order, because the concerns at each phase are distinct:

- **Preparation**: everything before touching a target, including scoping, rules of engagement, authorization, threat modeling, infrastructure setup, and reconnaissance planning.
- **Operational**: the engagement itself, covering execution flow, tradecraft, pacing, deconfliction, evidence capture, and staying within the agreed boundaries while adapting to what the target reveals.
- **Closure**: wrapping up, including cleanup of artifacts and access, findings analysis, reporting, and the handover that turns the operation into prioritized, actionable outcomes.

Splitting by phase matches how an operator actually thinks, moving through a timeline rather than jumping between techniques. A scoping decision and a reporting decision are different kinds of work with different inputs, so keeping them in separate phases prevents the practical discipline of running an engagement from being scattered across the technical categories. This is the category that keeps offensive work authorized, repeatable, and useful.

## References

- [PTES: Penetration Testing Execution Standard](http://www.pentest-standard.org/)
- [OWASP WSTG: Testing framework](https://owasp.org/www-project-web-security-testing-guide/latest/3-The_OWASP_Testing_Framework/)
