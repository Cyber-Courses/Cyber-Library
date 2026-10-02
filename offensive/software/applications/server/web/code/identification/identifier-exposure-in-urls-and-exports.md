---
title: "Identifier exposure: sequential IDs, over-sharing APIs, and leaky exports"
description: "How applications hand attackers the identifiers they need, through sequential or predictable IDs in URLs, over-sharing API responses, exports with hidden fields, and public profile and autocomplete endpoints."
keywords:
  - identifier exposure
  - sequential IDs
  - IDOR recon
  - excessive data exposure
  - user enumeration
  - data export
---

# Identifier exposure in URLs and exports

Applications constantly emit identifiers: user IDs, account numbers, order references, internal object keys. When those values are predictable or disclosed more widely than the interface implies, they become the raw material for enumeration and for access-control attacks. This page is about *collecting* identifiers, the step that precedes testing whether they can be accessed.

## Where identifiers leak

### Predictable IDs in URLs and parameters

- **Sequential integers**: `/user/1021`, `?order=5567`. Incrementing them enumerates the whole population and reveals counts and growth rate. A registration at two points in time brackets how many accounts exist between them.
- **Weakly random IDs**: short numeric IDs, timestamp-derived values, or `base64(email)` / `md5(id)` that reverse or regenerate. GUIDs are not automatically safe: a version-1 UUID encodes a timestamp and clock sequence plus a 48-bit node value, and where that node is a real MAC address (rather than the random node the spec also permits) consecutive IDs become predictable. Do not assume either property; capture several UUIDs and check the version nibble and whether the node and timestamp fields actually advance before relying on predictability.

### Over-sharing API responses

APIs frequently return more than the UI renders. A profile endpoint that shows a display name in the page may return email, phone, internal `user_id`, role, and tenant in the JSON. Inspect raw responses, not the rendered page:

- List and search endpoints that return full records for every row.
- Objects that embed related entities (an order carrying the full customer record).
- GraphQL, where a single query can request fields the UI never asks for, and where **introspection** maps every type and field available.

### Exports and documents

CSV, XLSX, and PDF exports are generated server-side and often include columns the web view hides, internal IDs, email addresses, status flags, or soft-deleted rows. Metadata in generated PDFs and images can carry usernames and paths. Request every export the app offers and diff its fields against the UI.

### Public and pre-auth endpoints

- **Profile pages** at `/u/<name>` confirm existence and often expose a numeric ID in the HTML or a linked avatar URL.
- **Autocomplete and typeahead** (`/api/users?q=jo`) return matching users to any caller, a bulk enumeration source.
- **Error messages and stack traces** that echo an ID or email.
- **Client-side bundles**: JavaScript and mobile apps embed API routes, field names, and sometimes test identifiers; read the bundle.

## Exploitation

Harvest, then enumerate. Scrape IDs from an autocomplete endpoint and expand a sequential range:

```bash
# pull user records across a sequential ID range, keep the ones that resolve
seq 1000 2000 | while read id; do
  curl -s "https://target/api/user/$id" \
    | jq -c 'select(.email) | {id, email, role}' 2>/dev/null
done
```

The resulting (id, email, role) set is directly usable: the emails feed [account enumeration](account-enumeration.md) and [credential stuffing](rate-limits-and-automation.md), and the IDs become test cases for access-control checks on every object type that keys on them. Over-sharing is itself the finding when a response or export discloses data the caller should not see; enumeration turns a single leak into a bulk one.

## Tools

- **Burp Suite**: Intruder to walk ID ranges; compare response fields; the sitemap to spot ID-bearing routes.
- **jq** / scripts: extract and diff fields from JSON responses and exports.
- **GraphQL tooling** (introspection queries, GraphQL Voyager) to map available fields.

## References

- OWASP API Security Top 10: [API1 Broken Object Level Authorization](https://owasp.org/API-Security/editions/2023/en/0xa1-broken-object-level-authorization/) and [API3 Broken Object Property Level Authorization / excessive data exposure](https://owasp.org/API-Security/editions/2023/en/0xa3-broken-object-property-level-authorization/)
- OWASP WSTG: [Testing for Account Enumeration (WSTG-IDNT-04)](https://owasp.org/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/03-Identity_Management_Testing/04-Testing_for_Account_Enumeration_and_Guessable_User_Account)
