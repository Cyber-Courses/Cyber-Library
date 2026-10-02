---
title: "JSP EL authorization bypass: forcing access decisions through scoped attributes"
description: "JSP unified EL reaches the session, request, and application scope maps through pageContext, so an injected ${...} expression reads and compares the attributes a role or flag check depends on."
keywords:
  - JSP EL authorization bypass
  - sessionScope EL
  - pageContext EL
  - JSP access control
  - EL injection
---

# Authorization bypass

JSP pages routinely gate markup and actions in EL: a conditional block renders on `${sessionScope.role == 'admin'}`, a link shows when `${user.admin}`, a tag is skipped unless `${requestScope.authorized}`. The unified EL exposes the scope maps and the `pageContext` as implicit objects, so an injected expression reads the same attributes the gate reads and, where the value flows into a session or application attribute, promotes an attacker-chosen value into the trusted scope.

## Reading the decision inputs

`sessionScope`, `requestScope`, and `applicationScope` are the attribute maps; `param` and `header` carry request input. An injected expression enumerates what the gate compares against:

```jsp
${sessionScope.role}
${sessionScope['isAdmin']}
${applicationScope.features}
```

Reading the session map turns a guessed attribute name into a confirmed one and shows the exact value a comparison expects.

## Forcing the comparison

When the injection point is the guard expression, resolve it to a constant truth so the gated branch always runs:

```jsp
${true}
${1==1}
${'admin' == 'admin'}
```

## Reaching the session through pageContext

`pageContext` exposes the live request, response, and session, so an expression navigates from it to the `HttpSession` and its attributes even when the scope maps are not named by the surrounding template:

```jsp
${pageContext.session.getAttribute('role')}
${pageContext.request.getSession().getAttribute('user')}
${pageContext.request.isUserInRole('admin')}
```

From `pageContext.request` the expression also reads `getParameter(...)` and `getRemoteUser()`, so an access value that the application copies from the request into a session attribute is observed, and the comparison that later trusts it is understood before it is targeted.

## Tools

- Manual testing with Burp Repeater; payloads crafted per engine.
- **Burp Intruder**: enumerate scope-map attribute names a guard compares against.

## References

- [Jakarta Expression Language Specification](https://jakarta.ee/specifications/expression-language/)
- [Jakarta Pages: Implicit EL Objects](https://jakarta.ee/specifications/pages/)
