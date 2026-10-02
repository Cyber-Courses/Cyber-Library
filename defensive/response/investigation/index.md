---
title: "Incident Response Investigation: Scoping and Evidence Gathering"
description: "How investigation scopes an incident, gathers evidence, and builds an understanding of what happened during active response."
keywords:
  - incident investigation
  - incident scoping
  - evidence gathering
  - root cause during response
  - forensic analysis
  - impact determination
---

# Investigation

Investigation is the work of understanding what happened during an active incident: how the adversary got in, what they touched, and how far the activity reaches. It builds the factual picture that response decisions depend on, scoping the incident so that containment and eradication address the whole problem rather than a fragment of it.

Within the Response phase, investigation runs in close partnership with containment. Early findings guide what to isolate, while continued investigation reveals the full extent of compromise. Acting on an incomplete picture risks missing footholds the adversary has established, so investigation and containment proceed together, each informing the other as the response unfolds.

In practice, investigation gathers and analyzes evidence from endpoints, logs, network data, and identity systems to reconstruct the adversary's actions. Responders identify affected systems and accounts, trace lateral movement, determine the initial access vector, and assess what data or capability was at risk. They preserve evidence carefully in case it is later needed for reporting or legal purposes. The output is a clear understanding of scope and impact that lets the team contain and eradicate with confidence, and that feeds the post-incident analysis once the event is resolved.

## References

- NIST SP 800-61, Computer Security Incident Handling Guide
- NIST SP 800-86, Guide to Integrating Forensic Techniques into Incident Response
