---
title: "Threat Hunting: Proactive Hypothesis-Driven Search"
description: "How threat hunting proactively searches telemetry for adversary activity that automated detection missed, guided by hypotheses."
keywords:
  - threat hunting
  - hypothesis-driven hunting
  - proactive detection
  - adversary behavior
  - hunt methodology
  - detection gaps
---

# Threat Hunting

Threat hunting is the proactive, hypothesis-driven search for adversary activity that existing detections did not catch. Rather than waiting for an alert, hunters form an idea about how an attacker might operate in the environment and then look through telemetry for evidence of it, assuming that a capable adversary may already be present.

Within the Detection phase, hunting complements automated detection. Rules and analytics cover known patterns well, but they leave gaps around novel techniques and subtle behavior. Hunting explores those gaps deliberately, using human judgment and curiosity to find what automation overlooks and to surface activity that blends into normal operations.

In practice, a hunt begins with a hypothesis grounded in threat intelligence, observed tactics, or knowledge of the environment. Hunters query collected data, pivot across hosts and identities, and confirm or reject the hypothesis with evidence. Findings feed back into detection: a successful hunt often becomes a new rule or analytic, turning a one-time search into lasting coverage. Hunting also builds analyst understanding of what normal looks like, which sharpens every other part of detection.

## References

- MITRE ATT&CK, Threat Hunting guidance
- SANS, Threat Hunting resources
