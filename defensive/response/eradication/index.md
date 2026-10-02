---
title: "Incident Response Eradication: Removing Adversary Presence"
description: "How eradication removes adversary presence and closes the entry vector so an incident cannot simply resume."
keywords:
  - incident eradication
  - removing adversary presence
  - closing entry vector
  - malware removal
  - persistence removal
  - threat elimination
---

# Eradication

Eradication is the removal of the adversary's presence from the environment and the closing of the vector that let them in. Where containment holds the problem in place, eradication eliminates it, ensuring that when systems return to service the threat does not simply resume.

Within the Response phase, eradication follows containment and precedes recovery. It depends on thorough investigation, because anything missed, an overlooked foothold, a backdoor, a compromised credential, can allow the adversary to return. Eradication is therefore as much about completeness as about action: the goal is to leave no path by which the same intrusion can continue.

In practice, eradication removes malware and attacker tooling, deletes unauthorized accounts and persistence mechanisms, revokes or resets compromised credentials, and remediates the weakness that enabled initial access. Heavily compromised systems are often rebuilt from known-good sources rather than cleaned in place, to be certain nothing remains. Responders verify that the adversary's access is truly gone before moving to recovery, often increasing monitoring to confirm the environment stays clean. Closing the entry vector, whether a vulnerability, a misconfiguration, or a stolen credential, turns eradication into lasting improvement rather than a temporary fix.

## References

- NIST SP 800-61, Computer Security Incident Handling Guide
- NIST SP 800-184, Guide for Cybersecurity Event Recovery
