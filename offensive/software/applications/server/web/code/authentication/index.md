---
title: "Attacking authentication in web application code: sessions, tokens, OAuth, SAML, and MFA"
description: An offensive guide to breaking how server-side code proves identity, credential, federated, MFA, session, and token handling, with the recon and attack methodology a pentester uses to reach account takeover.
keywords:
  - authentication attacks
  - account takeover
  - session management
  - JWT
  - OAuth
  - MFA bypass
---

# Authentication

**Authentication** is how server-side code establishes *who* the caller is before any authorization decision is made. For an attacker it is the highest-value target in the application: break it and you usually get **account takeover or full administrative access** outright, with no further chaining required. This section covers application-level weaknesses in that logic, credentials, federated logins, multi-factor checks, sessions, and tokens, as implemented in the target's own code and dependencies.

Authentication answers "who are you?"; [authorization](../access-control/index.md) answers "what may you do?". They break differently and are attacked differently, this subtree is strictly the former. When you can *become* another user, you rarely need to defeat access control at all.

## Where it breaks (your openings)

Authentication code is fragile because it stitches together cryptography, state, and third-party protocols. The recurring openings a pentester looks for:

- **Security decisions made on attacker-controlled input.** Token headers (`alg`, `kid`), redirect parameters, and the `Host` header are all client-modifiable, yet are frequently trusted *before* identity is proven. This is the richest seam, see [JWT algorithm confusion](jwt-algorithm-confusion.md).
- **Skipped checks in complex protocols.** OAuth, OIDC, and SAML have many mandatory-but-optional-looking parameters. Relying parties that code the happy path routinely drop `state`, `nonce`, audience, signature, or `redirect_uri` validation, each a foothold.
- **Home-grown primitives.** Predictable reset tokens, hand-rolled session IDs, and custom "remember me" cookies reintroduce solved problems you can exploit.
- **Missing throttling and lifecycle.** Login, OTP, and reset endpoints without rate limits invite brute force; sessions that never rotate or expire enable fixation and replay.

## Attack methodology

Work an authentication surface in this order:

1. **Map every entry point.** Password login, SSO/social login, API tokens, password reset, account invite, "remember me", step-up/MFA, and any legacy or mobile endpoint. Each is a separate trust boundary and a separate attack.
2. **Find the credential of trust.** Determine exactly what the server checks on each request, a session cookie, a JWT, a SAML assertion, and where it is verified. The verifier is your primary target; everything else is reconnaissance toward it.
3. **Attack the verifier.** Tamper with the token or assertion: swap algorithms and key-selection headers, strip or null signatures, replay across users and sessions, and test whether issuer, audience, expiry, and binding are actually enforced.
4. **Attack the flows around it.** For federated logins, go after `redirect_uri`/`state`/`nonce` handling and audience checks; for resets and invites, go after token entropy, lifetime, single-use, and host-header poisoning; for MFA, go after throttling, step ordering, and fallback paths.
5. **Escalate and persist.** Turn a foothold into impact: forge an admin identity, pivot across tenants, or capture long-lived tokens. Confirm whether privilege changes and logout actually invalidate prior sessions.

Stay within your authorized scope and rules of engagement, account-takeover testing manipulates real identities and must be contained to systems you are permitted to assess. Avoid locking out or destroying genuine accounts during brute-force and reset testing.

## Attack avenues

- **[JWT algorithm confusion](jwt-algorithm-confusion.md)**, forge JSON Web Tokens by abusing the attacker-controlled `alg` header (`none`, RS256→HS256 key confusion) or `jwk`/`jku`/`kid` key selection, so a token with chosen claims passes verification.
- **[OAuth redirect misconfiguration](oauth-redirect-misconfiguration.md)**, steal authorization codes or tokens through weak `redirect_uri` validation and missing `state`/`nonce`, including login CSRF against the relying party.
- **[Password reset and invite tokens](password-reset-and-invite-tokens.md)**, seize accounts via predictable, long-lived, reusable, or host-header-poisoned reset and invitation links.
- **[Session fixation](session-fixation.md)**, plant a known session identifier that is not regenerated on login, then ride the victim's authenticated session.
- **[Weak OTP and MFA throttling](weak-otp-and-mfa-throttling.md)**, brute-force or bypass second factors where one-time codes have weak entropy, no rate limiting, or broken ordering checks.

## Defenses you will encounter

Knowing the controls helps you spot when they are missing or misconfigured. Mature targets pin security-critical choices server-side (algorithms, key selection, redirect targets, audiences), use current vetted libraries, regenerate and expire sessions on privilege change and logout, give reset/OTP tokens short single-use lifetimes, and throttle every credential endpoint. Each missing or sloppily implemented control above is an entry on your test plan.
