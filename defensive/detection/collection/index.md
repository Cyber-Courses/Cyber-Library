---
title: "Detection Collection: Gathering Security Telemetry and Logs"
order: 3
description: "How defenders gather telemetry and logs from endpoints, network, identity, and cloud to make adversary activity visible."
keywords:
  - security telemetry collection
  - log collection
  - endpoint telemetry
  - network and cloud logs
  - detection data sources
  - visibility
---

# Collection

Collection is the practice of gathering the telemetry and logs that detection depends on. It spans endpoints, network traffic, identity and authentication systems, and cloud control planes, bringing raw signals into a place where they can be searched, correlated, and retained. Without reliable collection, later detection work has nothing to reason over.

Within the Detection phase, collection sits at the foundation. Discovery of adversary activity is only possible when the right events are captured in the first place, so defenders decide which sources matter, what fields to keep, and how long to store them. Gaps in coverage become blind spots that attackers can move through unseen.

In practice, collection involves deploying agents and forwarders, enabling audit and flow logging, normalizing formats, and centralizing data in a SIEM or data lake. Teams balance completeness against cost and noise, prioritizing high-value sources such as process creation, authentication events, DNS, and cloud API activity. Good collection is deliberate and reviewed over time, not simply everything that can be logged.

## References

- NIST SP 800-92, Guide to Computer Security Log Management
- MITRE ATT&CK, Data Sources
