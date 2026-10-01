---
title: "SOAP body parameter injection"
description: "Body children and wrapped document parameters built from input let type confusion, duplicate elements, and xsi:nil tricks change how the server unmarshals and authorizes a SOAP call."
keywords:
  - SOAP body injection
  - xsi:nil
  - duplicate element
  - unmarshaling confusion
  - wrapped literal
---

# Body parameter injection

The SOAP Body holds the operation's parameters as XML elements, which the server unmarshals into typed objects before the business logic runs. When those elements are built from partially controlled input, or when the unmarshaler is lenient about structure, an attacker changes the shape of the parameters in ways the schema and the authorization logic did not anticipate: nulling a field, duplicating an element, or confusing a type so the deserialized object differs from what was validated.

## Duplicate elements

XML allows a parameter element to appear more than once, and unmarshalers disagree on which wins (first, last, or an error). If a validation or authorization step reads one occurrence and the business logic reads another, the request is checked against a different value than it executes with:

```xml
<soap:Body>
  <Transfer xmlns="urn:Bank">
    <amount>1</amount>
    <toAccount>self</toAccount>
    <toAccount>victim</toAccount>   <!-- which one does the handler use? -->
  </Transfer>
</soap:Body>
```

## xsi:nil and missing fields

The `xsi:nil="true"` attribute marks an element as null even though it is present. Where a field is expected to carry a constraint (an owner, a tenant, a status), nulling it can drop a check that only runs when the field is populated, or coerce the object into a default the logic treats as privileged:

```xml
<GetDocument xmlns="urn:Docs" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
  <documentId>900</documentId>
  <ownerId xsi:nil="true"/>   <!-- bypass an owner-scoped filter that skips null owners -->
</GetDocument>
```

## Type confusion in unmarshaling

When an element's type is inferred or loosely bound (a wrapped document/literal parameter, an `xsi:type` override, or a field unmarshaled into a broad base type), supplying an unexpected type or structure can make the deserialized object differ from the schema-validated one. A field validated as a simple string may unmarshal into a complex type whose nested members reach code the simple case never did.

## Confirming the flaw

Send structurally valid envelopes that stress the unmarshaler: duplicate a security-relevant element with conflicting values, mark a constraint field `xsi:nil`, and override a type with `xsi:type`. A response that reflects the second duplicate, honors the nil by skipping a check, or processes the overridden type shows the unmarshaler and the validator saw different structures. Strict schema validation before business logic closes this, so leniency in how the body is parsed is what the test is probing. Raw XML parsing concerns such as entity expansion live in [XML processing](../../markup/xml-processing/index.md); this page is about how the unmarshaled parameters drive the operation.

## References

- [OWASP: Testing for SOAP](https://owasp.org/www-project-web-security-testing-guide/)
- [OWASP API Security Top 10](https://owasp.org/API-Security/editions/2023/en/0x11-t10/)
