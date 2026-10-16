---
title: "Authentication and Authorization: Proving Identity, Then Granting Access"
description: "The paired security processes that first establish who a principal is and then decide what that principal is permitted to do."
keywords:
  - authentication and authorization
  - authentication factors
  - access control
  - identity verification
  - broken access control
  - confused deputy
---

# Authentication and authorization

**Authentication and authorization are two separate decisions that happen in sequence, and confusing them is one of the oldest sources of security bugs.** Authentication answers "who are you?" by verifying that a principal (a user, service, or device) is who it claims to be. Authorization answers "what are you allowed to do?" by deciding whether that verified principal may perform a specific action on a specific resource. The first establishes identity; the second gates access.

## Two questions, asked in order

For anything gated by identity, the order is fixed: a system must know who a principal is before it can sensibly decide what that principal may do, so authentication comes first and authorization follows. Not everything is gated that way, though. A public page can be read with no identity at all, and a request can be denied outright without authenticating anyone; it is only when access depends on who you are that authentication has to come first. The two are often shortened to "authN" and "authZ" to keep them distinct in speech and in code.

A useful test: authentication asks whether you are who you claim, and the check is the same no matter what you go on to do; authorization asks whether your established identity may take a specific action on a specific resource, so its answer varies with the resource, the action, and often the identity itself. Deciding that Alice may export payroll while Bob may not is authorization, not authentication, even though the outcome changes with who logged in. A valid login that then reaches someone else's records is likewise an authorization failure, not an authentication one, even though both happened in the same request.

## The three factors of proof

Authentication rests on evidence the claimant can produce. These fall into three classic categories, and combining categories is what makes multi-factor authentication strong.

| Factor | What it is | Examples |
|---|---|---|
| Something you know | A secret held in memory | Password, PIN, passphrase |
| Something you have | A physical or cryptographic token | Security key, phone app, smart card |
| Something you are | A biometric trait | Fingerprint, face, iris |

Two passwords are not multi-factor, because they draw on the same category and fall to the same theft. A password plus a hardware key resists an attacker who has only phished the password.

## Walking through a single sign-in

Consider a user opening an internal dashboard:

1. **Claim.** The browser presents a username and asks to log in.
2. **Authenticate.** The server checks the password hash, then prompts for a one-time code from the user's phone. Both factors pass, so identity is established.
3. **Issue proof.** The server hands back a signed session token that encodes the verified identity.
4. **Authorize.** When the user clicks "export payroll," the server reads the token's identity, looks up that user's roles, and finds no finance permission. The request is denied.

Steps 1 to 3 are authentication; step 4 is authorization. A system can get the login perfectly right and still leak data if step 4 is weak.

## The confused deputy and broken access control

The most common failures sit in authorization. **Broken access control** means the server trusts the client to stay inside its lane: changing an ID in a URL reaches another account's data because the server authenticated the session but never re-checked ownership. The **confused deputy** is subtler: a trusted program is tricked into using its own higher privileges for a less privileged caller, because it acts on its own authority instead of carrying over and constraining the caller's. The two differ in cause, broken access control fails to re-check a session while the confused deputy fails to pass along the caller's limited authority, but both let through an action the requester's own permissions would have refused.

## Where it connects to the other principles

Authorization is the point where [least privilege](least-privilege.md) is actually enforced, request by request. [Zero trust](zero-trust.md) refuses to treat one successful login as lasting permission and re-verifies each request instead. And because authentication binds actions to a named identity, it is the foundation for [accountability](accountability.md): logs are only meaningful when you can trust who performed each action.

## References

- [NIST SP 800-63B, Digital Identity Guidelines: Authentication](https://pages.nist.gov/800-63-3/sp800-63b.html)
- [OWASP Top 10: A01 Broken Access Control](https://owasp.org/Top10/A01_2021-Broken_Access_Control/)
- [OWASP Authentication Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html)
- [NIST Glossary: Authentication](https://csrc.nist.gov/glossary/term/authentication)
- [MITRE CWE-862: Missing Authorization](https://cwe.mitre.org/data/definitions/862.html)
