---
title: "Detection Engineering: Building and Maintaining Detections"
order: 6
description: "How detection engineering designs, tests, and maintains the rules and analytics behind reliable detection, treating detections as code."
keywords:
  - detection engineering
  - detection as code
  - detection rules
  - analytics testing
  - rule lifecycle
  - detection maintenance
---

# Engineering

Detection engineering is the disciplined work of designing, testing, and maintaining the rules and analytics that make detection reliable. It treats detections as engineered products with a lifecycle, rather than one-off searches, applying software practices so that coverage is deliberate, repeatable, and durable.

Within the Detection phase, engineering is what keeps analysis effective over time. Environments change, adversary techniques evolve, and untended rules decay into noise or silence. Detection engineering manages that drift by versioning detections, validating them against realistic activity, and retiring those that no longer earn their place.

In practice, the discipline is often described as detection-as-code: rules live in version control, change through review, and are tested in pipelines before deployment. Engineers map detections to known techniques, write them to be precise and explainable, and measure their false-positive and true-positive behavior. They document intent and data dependencies so a detection can be understood and maintained by others. The result is a managed catalog of detections whose quality and coverage can be reasoned about, rather than an opaque pile of rules no one dares to touch.

## References

- MITRE ATT&CK, Detections and Analytics
- SANS, Detection Engineering resources
