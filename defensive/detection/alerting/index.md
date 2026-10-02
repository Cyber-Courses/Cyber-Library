---
title: "Detection Alerting: Routing Findings and Reducing Fatigue"
description: "How alerting routes detection findings with context to the right responders while tuning to reduce alert fatigue."
keywords:
  - security alerting
  - alert routing
  - alert fatigue
  - detection tuning
  - alert enrichment
  - triage prioritization
---

# Alerting

Alerting is how detection findings reach the people who can act on them. It routes signals produced by analysis to the right responders, carrying enough context for a recipient to understand what happened and decide what to do next. An alert without context is a question; an alert with context is a starting point.

Within the Detection phase, alerting is the bridge to response. It sits after analysis has judged an event worth attention and before investigation begins. The quality of alerting shapes how quickly and accurately a team can act, so defenders care not only about catching activity but about presenting it clearly and sending it to the correct destination.

In practice, alerting involves enrichment with asset, identity, and threat context, severity scoring, deduplication, and routing to queues, chat, or ticketing systems. A central concern is alert fatigue: too many low-value alerts dull attention and let real incidents slip past. Teams tune detections, suppress known-benign patterns, and prioritize by risk so that what reaches an analyst is worth their time. Continuous tuning keeps the signal strong as the environment changes.

## References

- NIST SP 800-61, Computer Security Incident Handling Guide
- SANS, Security Operations Center resources
