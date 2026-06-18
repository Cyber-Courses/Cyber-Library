---
title: "Attacking session fixation: planting a known session ID and missing rotation on login"
description: How a pentester fixes a victim's session identifier before login and rides the authenticated session—planting vectors, the permissive vs strict distinction, a step-by-step walkthrough, detection, and remediation.
keywords:
  - session fixation
  - session management
  - cookie tossing
  - session hijacking
  - JSESSIONID
---

# Session fixation

A session-hijacking technique that needs no stolen session ID, because the attacker **chose it in advance**. The attacker plants a known session identifier in the victim's browser, the victim authenticates, and—because the application fails to issue a new identifier at login—the now-authenticated session still carries the attacker-known value. The attacker rides it into the victim's account.

> **Scope.** Authorized testing only. Fixation requires interacting with a victim's browser session; confine it to systems and accounts in scope.

## Overview

Session fixation is the inverse of session hijacking: in hijacking the attacker exfiltrates a server-generated ID *after* login; in fixation the attacker plants an ID *before* login and waits. The single root cause is that **the session identifier is not regenerated on the privilege change at authentication**—the same value spans the unauthenticated → authenticated boundary.

Two server behaviors set the attacker's effort (Kolšek's original distinction):

- **Permissive** servers accept arbitrary client-supplied IDs and create a session for them (PHP defaults this way). The attacker simply invents an ID such as `1234`.
- **Strict** servers only honor IDs they generated. The attacker must first request the app unauthenticated to obtain a real "trap" session ID, then keep it alive (periodic requests defeat idle timeout) until the victim logs in.

## Exploitation

### Step 1 — obtain or choose the trap ID

Permissive target: pick any value. Strict target: hit the app unauthenticated, capture the issued `Set-Cookie: SESSIONID=…`, and keep it warm with periodic requests until used. The absolute timeout and server restarts bound the attack window.

### Step 2 — plant it in the victim's browser

Same-origin rules mean the attacker's own domain cannot set a cookie for the target, so planting uses a target-origin vector:

- **URL parameter** (where the app accepts session IDs from the query, e.g. Java URL rewriting): `https://app.site/login;jsessionid=1234` or `?sessionid=1234`. Simplest, but visible and easily logged.
- **Client-side script via XSS** (no `HttpOnly`): reflect `document.cookie="sessionid=1234"` through a target-origin sink; add `Expires=` to upgrade it to a durable cookie and extend the window.
- **`<meta http-equiv="Set-Cookie">`** injected into a target-origin page.
- **Subdomain cookie tossing**: from a weaker sibling host, set a cookie scoped to the parent domain so the browser sends it to the target—`document.cookie="sessionid=1234;domain=.site"`. A single XSS or header-injection on *any* subdomain compromises the whole domain.
- **Response-header injection** (HTTP response splitting, or a misbehaving proxy/CDN) to emit `Set-Cookie` directly.

### Step 3 — ride the authenticated session

Wait for the victim to log in. Because the ID was not rotated, the authenticated session still carries `1234`; access it with `Cookie: sessionid=1234`.

### Confirming it as a pentester (black-box)

Record the cookie issued **pre-login**, submit valid credentials, and inspect the response: if the server returns the **same** session identifier with no fresh `Set-Cookie`, fixation is confirmed. WSTG's forced-cookie procedure uses two accounts—save the victim's pre-login cookies (skip `__Host-`/`__Secure-`-prefixed ones), authenticate, restore the saved cookies, and check whether a protected action still works under the victim's identity.

## Related session-management weaknesses

- **No server-side invalidation on logout**—clearing only the client cookie leaves the fixed ID live; a real fix deletes the session server-side (and ideally "logs out everywhere").
- **Overly long or missing absolute timeout**—lets the attacker keep a trap session alive and ride an entered one.
- **Predictable / low-entropy IDs**—let an attacker guess valid trap IDs even on strict servers.
- **Missing `Secure`/`HttpOnly`/`SameSite`**—don't cause fixation but enable the planting step (script access, network injection, cross-site delivery).

## Detection (code review)

The audit question: does the framework issue a **new** session ID at authentication and reject client-supplied IDs?

- **Java/JEE**: `request.changeSessionId()` (Servlet 3.1+) or `invalidate()` + `getSession(true)`. Suspect: reusing a session across login; `;jsessionid` URL rewriting; Spring Security `sessionFixation().none()`. Grep `getSession`, `changeSessionId`, `invalidate`, `jsessionid`.
- **PHP**: `session_regenerate_id(true)` immediately after auth; flag its absence and `session.use_strict_mode=0`. Grep `session_regenerate_id`, `use_strict_mode`, `session_id(`.
- **ASP.NET**: the built-in `ASP.NET_SessionId` is not auto-rotated—rely on auth-cookie reissue (`Session.Abandon` + re-auth). Grep `Session.Abandon`, `SignOut`.
- **Express/Node**: call `req.session.regenerate()` in the login callback before establishing the authenticated session (Passport's `req.login` does not regenerate by default in older setups). Grep `session.regenerate`, `req.login`.

Across stacks, flag any path that reads the session ID from `GET`/`POST` parameters.

## Remediation

1. **Regenerate the session ID on every authentication and privilege change**, and **invalidate the old session server-side**.
2. **Reject client-supplied session IDs** (PHP strict mode; never read the ID from URL/POST); on strict servers issue authenticated IDs only after successful auth.
3. **Cookie flags**: `Secure`, `HttpOnly`, `SameSite`, tight `Domain`/`Path`, and the `__Host-` prefix to block cookie tossing.
4. **Idle and absolute timeouts**, server-side destruction on logout, and strong high-entropy IDs.

## Tools

- **Burp Suite** — observe `Set-Cookie` across the login boundary; the session-handling rules and macros to script the pre-/post-login comparison.

## References

- OWASP — [Session fixation](https://owasp.org/www-community/attacks/Session_fixation) and the [Session Management Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Session_Management_Cheat_Sheet.html)
- OWASP WSTG — [Testing for Session Fixation (WSTG-SESS-03)](https://owasp.org/www-project-web-security-testing-guide/stable/4-Web_Application_Security_Testing/06-Session_Management_Testing/03-Testing_for_Session_Fixation.html)
- Mitja Kolšek / ACROS Security — [Session Fixation Vulnerability in Web-based Applications](https://acrossecurity.com/papers/session_fixation.pdf) (the seminal paper)

## See also

- [Authentication (parent)](index.md)
- [Markup injection / XSS](../injection/markup/index.md) — a common planting vector for the fixed cookie.
- [HTTP response splitting](../injection/http/http-response-splitting.md) — header injection used to set the cookie.
