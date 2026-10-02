---
title: "JSF EL authorization bypass: forcing access decisions through scoped attributes"
description: "JSF unified EL reaches the session, request, and application scope maps, so an injected #{...} expression reads and overwrites the attributes a role or flag check depends on."
keywords:
  - JSF EL authorization bypass
  - sessionScope EL
  - unified EL implicit objects
  - JSF access control
  - EL injection
---

# Authorization bypass

JSF access decisions are frequently expressed in EL itself: a `rendered` attribute, a navigation condition, or a secured component gates on `#{sessionScope.role == 'admin'}` or `#{user.admin}`. Because the unified EL exposes the scoped attribute maps as implicit objects, an injected expression reads those same values and, where the evaluation context is writable, sets them. The authorization logic and the attacker then share one namespace.

## Reading the decision inputs

The implicit objects `sessionScope`, `requestScope`, and `applicationScope` are maps of the corresponding attributes, and `param` holds request parameters. An injected expression reads whatever the gate reads:

```jsp
#{sessionScope.role}
#{sessionScope['isAdmin']}
#{applicationScope.featureFlags}
```

Reading the session map enumerates the exact attribute names and values an access check compares against, which turns a guessed flag into a known one.

## Forcing the comparison

When the gate compares a scoped attribute, an expression that resolves to the expected value satisfies it directly. If the injection point is the guard expression itself, replace the comparison with a constant truth:

```jsp
#{true}
#{1==1}
```

Where the application evaluates a value expression against a writable EL context (a `setValue` path, or an assignment-capable EL 3.0 context), the scoped attribute is overwritten so a later, separate check reads the attacker's value:

```jsp
#{sessionScope.isAdmin = true}
```

## Reaching the session through the context

The faces context exposes the live request and session, so an expression navigates from the context to the `HttpSession` and its attributes even when the scope maps are not directly named by the template:

```jsp
#{facesContext.externalContext.sessionMap['role']}
#{facesContext.externalContext.request.getSession().getAttribute('user')}
```

From the same `externalContext`, `getRequestParameterMap()` and the session map let the expression copy an attacker-chosen value into a session attribute that downstream code trusts, so a privilege held only in the request is promoted into the session.

## Tools

- Manual testing with Burp Repeater; payloads crafted per engine.
- **Burp Intruder**: enumerate scoped attribute names an access check compares against.

## References

- [Jakarta Expression Language Specification](https://jakarta.ee/specifications/expression-language/)
- [Jakarta Faces: ExternalContext](https://jakarta.ee/specifications/faces/)
