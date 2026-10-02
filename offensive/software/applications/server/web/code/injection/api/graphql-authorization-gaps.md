---
title: "GraphQL authorization gaps: resolver-level BOLA, nested traversal, and dataloader batching"
description: Exploiting GraphQL APIs where the HTTP route is authenticated but individual resolvers never scope the object to the caller, reaching other tenants' rows through nested selections, mutations, and dataloader-batched fields.
keywords:
  - GraphQL
  - BOLA
  - IDOR
  - broken object level authorization
  - resolver authorization
  - dataloader
---

# GraphQL authorization

A GraphQL endpoint usually sits behind a single authentication gate: a middleware that verifies the session or token and rejects anonymous callers. That gate answers *"is this a valid user?"*, it says nothing about *"is this user allowed to see this particular object?"* Because every field is resolved by its own function, object-level authorization has to be enforced **per resolver**, on the specific row being loaded. Where it is not, an authenticated attacker simply asks for objects that are not theirs, and the server returns them. This is **broken object-level authorization (BOLA / IDOR)** expressed through the resolver graph.

## Overview

The core mismatch: authentication is enforced **once at the edge**, authorization must be enforced **many times in the graph**. A resolver that takes an `id` argument and loads that row, without checking the row belongs to the caller's tenant, is a direct object reference under a different name:

```graphql
query { invoice(id: "INV-2041") { total customerEmail lineItems { sku } } }
```

If `invoice` resolves straight from the `id` with no ownership predicate, incrementing or guessing identifiers walks the whole table. The underlying root cause is shared with REST IDOR and the object-level patterns in **[Access control](../../access-control/index.md)**; GraphQL only changes the surface, giving the attacker a typed map (via introspection) and many more reachable fields per request.

## Enumerating the attack surface

Pull the schema first (see [GraphQL query shape](graphql-query-shape-abuse.md) for introspection). From the types, catalogue:

- **Queries and mutations that take an object identifier** (`id`, `uuid`, `slug`, `accountId`), each a candidate for direct reference.
- **Fields that return other objects**, relationships let you reach a protected type *through* an unprotected parent.
- **Mutations hidden in the same schema as reads**, `updateUser`, `deleteInvoice`, `setRole`, which frequently receive far less authorization scrutiny than the queries.

Node-style globally unique IDs are often base64 of `Type:id`; decoding them reveals the format and lets you forge references to adjacent objects:

```
echo -n 'VXNlcjoxMDI0' | base64 -d      # => User:1024  →  try User:1025, Invoice:… etc.
```

## Exploitation patterns

### Direct object reference

The simplest case: a top-level resolver returns any object by id regardless of owner. Authenticate as a low-privilege user, then request another user's object by id and confirm the data comes back. Use two test accounts you control, fetch account A's object id while logged in as B, to prove cross-tenant read rather than guessing blindly.

### Nested traversal around the gate

Even when the top-level query *is* guarded, a nested field reached through an authorized parent often is not. The parent passes the object-level check; the child resolver trusts that it was reached legitimately and loads a related row with no check of its own:

```graphql
query {
  me {                         # authorized: my own user
    organization {             # my org
      members {                # every member of the org...
        privateNotes { body }  # ...including notes that should be owner-only
      }
    }
  }
}
```

The attacker never references a foreign id directly; they ride a chain of relationships from an object they *are* allowed to see into one they are not. Deeply nested traversals are where resolver-level BOLA most often hides, because reviewers check the entry point and assume the rest inherits its protection.

### Mutations

Mutations are state-changing and disproportionately under-guarded. Test every mutation that accepts an identifier against objects owned by another account:

```graphql
mutation { updateInvoice(id: "INV-2041", input: { status: PAID }) { id status } }
mutation { addOrgMember(orgId: "ORG-7", userId: "me", role: ADMIN) { ok } }
```

A mutation that succeeds against an object you do not own, or that lets you set a privileged field (`role`, `isAdmin`, `ownerId`) the server should control, is a direct finding. Mass-assignment through the `input` object, setting fields the client should not be able to, often rides alongside the missing ownership check.

### Dataloader batching masking checks

The **dataloader** pattern batches many id lookups into one backend query to avoid the N+1 problem. The danger is that authorization written as a per-request predicate can be bypassed when the dataloader loads rows by id in bulk and hands each resolver its row *without* re-applying the per-object check. Nested fields that fan out across many parents (a list of orders, each loading its customer) are the place to probe: request a collection whose elements belong to several tenants and see whether the batched loader returns rows it should have filtered. The batching that makes the API fast is exactly what makes a missing scope check invisible until foreign rows appear in the response.

### Field-level leakage

Authorization can be correct at the object level yet wrong at the **field** level: the object is yours to see, but a specific field (another user's email on a shared thread, an internal cost field, a password-reset token) should be restricted and is not. Select every field the schema offers on each object and compare what returns across accounts of different privilege; fields that appear only when they shouldn't are the finding.

## Confirming a finding

Use at least two controlled accounts in different tenants. For each candidate query or mutation, run it authenticated as the account that should **not** have access and verify the foreign object's data is returned or mutated. A clean proof pairs the request with the second account's own view of the same object id, showing they are genuinely separate and that the boundary was crossed.

## Tools

- **[Burp Suite](https://portswigger.net/burp)** with **[InQL](https://github.com/doyensec/inql)** and **[Autorize](https://github.com/Quitten/Autorize)** to replay each operation under a second, lower-privileged session and diff the responses automatically.
- **[GraphQL Voyager](https://github.com/graphql-kit/graphql-voyager)** to map relationships and plan nested-traversal paths from authorized parents to protected children.
- **[clairvoyance](https://github.com/nikitastupin/clairvoyance)** to reconstruct the schema when introspection is disabled, so object and mutation arguments are still enumerable.

## References

- [OWASP API Security Top 10: API1 Broken Object Level Authorization](https://owasp.org/API-Security/editions/2023/en/0xa1-broken-object-level-authorization/)
- [OWASP: GraphQL Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/GraphQL_Cheat_Sheet.html)
- [PortSwigger Web Security Academy: GraphQL API vulnerabilities](https://portswigger.net/web-security/graphql)
- [CWE-639: Authorization Bypass Through User-Controlled Key](https://cwe.mitre.org/data/definitions/639.html)
