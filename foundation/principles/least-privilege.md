---
title: "Least Privilege: Granting Only the Access That Is Needed"
description: "The security principle that every user, process, and system should hold only the permissions required for its task, for only as long as it needs them."
keywords:
  - least privilege
  - principle of least privilege
  - access control
  - privilege management
  - just-in-time access
  - security principles
---

# Least privilege

The **principle of least privilege** states that every user, program, and system should operate with the smallest set of permissions it needs to do its job, and nothing more. The logic is simple: access that does not exist cannot be misused, whether by a careless employee, a compromised account, or a piece of malicious code running under someone's identity.

## Saltzer and Schroeder's one-line rule

The principle was framed in 1975 by Jerome Saltzer and Michael Schroeder as one of eight design principles for protecting information in computer systems. Their wording is short: "Every program and every user of the system should operate using the least set of privileges necessary to complete the job."

The paper gives three reasons that still hold today:

- **Damage control.** The rule "primarily" limits the harm that can follow from an accident or error.
- **Fewer interactions to audit.** If only a few programs hold a privilege, then only those few need to be examined when that privilege is misused.
- **A map for firewalls.** The authors note that least privilege tells you where internal barriers belong, and they cite the military "need-to-know" rule as an example of the same idea.

The same paper also anticipates a modern habit: a single person may want to act under a second, less powerful principal when doing unrelated work, so that a mistake cannot touch the valuable information their main identity can reach.

## Who, what, and for how long

Applying the principle means narrowing access along three separate dimensions. Most real techniques tighten one of them:

| Dimension | Question to ask | Typical technique |
|---|---|---|
| Who | Does this identity need access at all, and is it tied to one accountable owner? | Role-based access, so permissions follow job functions and are lost when the role changes; no shared generic accounts |
| What | Which resources and which actions, exactly? | Scoped credentials for applications, scripts, and cloud workloads, limited to the specific resources and operations they use |
| When | Is the access needed now, or permanently? | Just-in-time and time-bound elevation that is granted on request for a defined window and expires automatically |

A fourth technique cuts across all three: **separate privileged accounts**. Administrators do daily work such as email and browsing from a standard account, and switch to a distinct, more powerful account only for administrative tasks. This is the "second principal" idea from 1975 in its everyday form.

## Following one stolen credential

Consider an attacker who phishes the credentials of a developer whose account is used by a deployment pipeline.

1. **Broad version.** The account holds an administrator role across the whole cloud environment. The attacker can read every database, create new users for persistence, and disable logging. One phishing email becomes a full compromise.
2. **Narrow version.** The pipeline uses its own service identity that can only push images to one repository and update one application. The developer's personal account has read-only production access, with write access available just in time after approval.
3. **What the attacker now faces.** The stolen credential reaches a small slice of the environment. To go further, the attacker must find and exploit additional weaknesses, which takes time and generates noise that defenders can detect.
4. **What investigators see.** Because each identity has a clear, minimal purpose, an action outside that purpose (such as a read-only account attempting a write) stands out in the logs and is easy to attribute.

The same narrowing protects against honest mistakes. Many outages and data losses come from someone deleting the wrong resource or changing a production setting with access they did not need for the task at hand.

## How privilege creeps back

Least privilege is not a one-time configuration. Access tends to grow unless something actively shrinks it:

- **Privilege creep:** people change teams or projects and keep the access they had before, slowly accumulating far more than any single role requires.
- **Convenience grants:** broad roles such as "administrator" or wildcard cloud policies are handed out because defining precise permissions takes effort.
- **Standing and shared credentials:** high-privilege credentials that never expire, and generic accounts used by several people, undermine both minimal access and traceability.
- **Forgotten non-human identities:** service accounts, API keys, and automation tokens are often overprivileged and rarely reviewed, even though they can outnumber human users.

The counterweight is the **regular access review**: permissions are periodically compared against actual use and need, and unused or excessive rights are removed. NIST SP 800-53 builds this into its least privilege control family (AC-6), which includes reviewing assigned privileges to confirm they are still needed.

## Where it meets the other principles

Least privilege rarely works alone. Authorization, covered in [authentication and authorization](authentication-and-authorization.md), is the point where it is actually enforced, request by request. [Zero trust](zero-trust.md) verifies every request and pairs naturally with granting each one only the minimum access it needs, which is why just-in-time access appears in zero trust maturity models. Within [defense in depth](defense-in-depth.md), tight permissions are the layer that keeps a single compromised account from exposing the whole system. Finally, [minimization](minimization.md) applies the same reasoning to data and attack surface: keep less, expose less, and there is less to lose.

## References

- [Saltzer and Schroeder, The Protection of Information in Computer Systems (1975)](https://www.cs.virginia.edu/~evans/cs551/saltzer/)
- [NIST Glossary: Least Privilege](https://csrc.nist.gov/glossary/term/least_privilege)
- [NIST SP 800-53 Rev. 5, Security and Privacy Controls (AC-6 Least Privilege)](https://csrc.nist.gov/pubs/sp/800/53/r5/upd1/final)
- [CISA: Zero Trust Maturity Model](https://www.cisa.gov/zero-trust-maturity-model)
- [OWASP Top 10 Proactive Controls, C1: Implement Access Control](https://top10proactive.owasp.org/the-top-10/c1-accesscontrol/)
- [OWASP Authorization Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Authorization_Cheat_Sheet.html)
