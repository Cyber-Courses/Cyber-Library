---
title: "Authentication and Authorization: Proving Identity, Then Granting Access"
description: "The paired security functions that first establish who a principal is and then decide what that principal is permitted to do."
keywords:
  - authentication
  - authorization
  - access control
  - identity verification
  - multi-factor authentication
  - security principles
---

# Authentication and authorization

**Authentication and authorization** are two distinct steps that work together to control access. Authentication answers the question "who are you?" by verifying that a principal (a user, service, or device) is who it claims to be. Authorization answers the separate question "what may you do?" by deciding which actions and resources that verified principal is allowed to reach. The order matters: a system establishes identity first, then applies the rules that govern what that identity can touch. Blurring the two, or skipping one, is a common source of serious security failures.

## Two questions, two different failures

A useful picture is a building: authentication is the front door that keeps unknown or impersonated people out, and authorization is the set of locked hallways that keeps a verified visitor from wandering into rooms that are none of their business. Because the two steps answer different questions, they also break in different ways, and confusing them leads to fixing the wrong problem.

| | Authentication | Authorization |
|---|---|---|
| Question | Who is making this request? | Is this principal allowed to do this, to this resource? |
| Typical evidence | Passwords, tokens, passkeys, biometrics | Roles, attributes, ownership, policy rules |
| What a failure looks like | An attacker logs in with a guessed, phished, or reused password | An ordinary user reads an administrator's records |
| What it enables | Accountability and meaningful audit logs | Least privilege in practice |

Stolen credentials and broken access checks are among the most frequent and damaging weaknesses in modern systems. OWASP ranks Broken Access Control first in its Top 10 for web applications, in both the 2021 and 2025 editions.

## Proving identity: factors and how they are defeated

Authentication evidence is traditionally grouped into three factors. Each has its own weak spot, which is why combining them matters.

- **Something you know:** passwords and PINs. They can be guessed, phished, or leaked from one service and replayed on another when people reuse them.
- **Something you have:** hardware tokens, authenticator apps, or passkeys bound to a device. They resist remote guessing, but the device can be lost or the user tricked into approving a login.
- **Something you are:** fingerprints, faces, and other biometrics. They are convenient, but they cannot be changed if compromised, so they usually unlock a local key rather than travel over the network.

**Multi-factor authentication** combines two or more of these so that one stolen secret is not enough to impersonate someone. **Federated identity and single sign-on** move the work to a trusted identity provider: it authenticates the user once, and relying applications accept its signed assertion instead of each keeping its own password database.

## Deciding access: from roles to policies

Once identity is established, authorization rules must be written in a form people can maintain and review. Three families of models are common:

1. **Role-based access control (RBAC):** permissions attach to roles such as "billing clerk" or "auditor", and users receive roles. Simple to reason about, but roles tend to multiply over time.
2. **Attribute-based access control (ABAC):** decisions weigh attributes of the user, the resource, and the context (department, data classification, time of day).
3. **Policy-based access control:** rules are expressed as centrally managed policies, evaluated by a decision engine and applied consistently across services.

Whatever the model, enforcement belongs on the server and runs for every request. The user interface can hide a button for convenience, but it is never the place where access is actually decided. Authorization is also the layer where [least privilege](least-privilege.md) becomes real: each identity receives only what its task requires.

## Tracing one request: "show invoice 1042"

Consider a signed-in user, Alice, whose browser asks an accounting app for invoice 1042. Following the request step by step shows where each check belongs and what goes wrong when one is missing.

1. **Alice's session token arrives.** The server validates it and concludes "this is Alice." That is authentication, and it is only as strong as the login behind it: if Alice reused a password that leaked elsewhere, an attacker can simply log in as her and receive a perfectly valid token of their own.
2. **The server looks up invoice 1042.** Knowing who Alice is says nothing yet about whether this invoice is hers.
3. **The ownership check.** The server must confirm that invoice 1042 belongs to Alice's account, or that her role permits viewing it. Skipping this step is a broken object-level check: by changing the number in the URL, Alice could read every customer's invoices (an insecure direct object reference).
4. **The action check.** Viewing and deleting are different permissions. A system that only hides the "delete" button, while leaving the delete endpoint callable, has gatekept on the client side.
5. **The decision is logged against Alice's identity.** This is what makes [accountability](accountability.md) possible: a log entry is only meaningful when it is tied to a reliably authenticated principal.

The scenario also exposes a quieter failure: **stale permissions**. If Alice moved from billing to marketing last quarter and her old role was never removed, every check above still passes, correctly applying rules that should no longer exist.

## Checking again on every request

Modern architectures push both steps further from a one-time login. [Zero trust](zero-trust.md) treats every request as untrusted and re-verifies identity and permissions rather than relying on network location, and [defense in depth](defense-in-depth.md) treats identity and access checks as one layer among several, so that a single failure does not expose the whole system. Note that "authorization" also has a legal and ethical sense, the permission to test or access a system at all, which is covered in [authorization and consent](../ethics-and-philosophy/authorization-and-consent.md).

## References

- [NIST SP 800-63-4 Digital Identity Guidelines](https://pages.nist.gov/800-63-4/)
- [NIST Glossary: Authentication](https://csrc.nist.gov/glossary/term/authentication)
- [OWASP Top 10:2025, A01 Broken Access Control](https://owasp.org/Top10/2025/A01_2025-Broken_Access_Control/)
- [OWASP Authentication Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html)
- [OWASP Authorization Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Authorization_Cheat_Sheet.html)
- [MITRE CWE-862: Missing Authorization](https://cwe.mitre.org/data/definitions/862.html)
