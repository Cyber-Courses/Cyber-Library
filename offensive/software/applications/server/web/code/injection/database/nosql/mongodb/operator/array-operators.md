---
title: "MongoDB array-operator injection with attacker-supplied arrays"
order: 2
description: "Injecting $in, $nin, $all, and $elemMatch from attacker-controlled arrays widens matches, enumerates candidate values, and probes array fields."
keywords:
  - array operators
  - $in
  - $nin
  - $all
  - $elemMatch
  - MongoDB injection
---

# Array operators

The array operators (`$in`, `$nin`, `$all`, `$elemMatch`) take an array as their argument, so they are the natural payload when a parser or JSON body lets an attacker supply an array where the application expected a scalar. They widen a match set, enumerate candidate values in one request, and reach into array-valued fields.

The precondition is the usual one: the value reaches the filter as an attacker-controlled array or object, not a cast string.

## $in and $nin to widen or invert a match

`$in` matches a field against any value in an array; `$nin` matches any document whose field is in none of them. Supplying a broad or inverted set turns a narrow lookup into a wide one:

```json
{"role": {"$in": ["admin", "superuser", "root", "operator"]}}
{"status": {"$nin": ["banned"]}}
{"_id": {"$in": [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]}}
```

`$nin` with a short exclusion list returns nearly everything, slipping past a filter meant to select one record. Through a nested parser the array is built with repeated bracket keys:

```
role[$in][]=admin&role[$in][]=root
_id[$nin][]=0
```

`$in` also enumerates: supply a batch of candidate values for a guarded field (role names, coupon codes, feature flags) and a single response tells you whether any of them matches, which narrows the set for the next request.

## $all against array-valued fields

`$all` requires a field (itself an array) to contain every element you list. Against fields like `roles`, `permissions`, or `tags` it tests membership precisely:

```json
{"roles": {"$all": ["admin"]}}
{"permissions": {"$all": ["read", "write", "delete"]}}
```

An empty array passed to `$all` matches no documents, a handy false branch when building an oracle; a single-element `$all` is an equality test on array membership that confirms whether a target holds a given role.

## $elemMatch to query inside array elements

`$elemMatch` matches documents where at least one element of an array field satisfies a sub-query, and that sub-query can itself carry injected operators. It reaches into nested documents stored in arrays:

```json
{"accounts": {"$elemMatch": {"balance": {"$gt": 10000}}}}
{"sessions": {"$elemMatch": {"token": {"$regex": "^ey"}}}}
```

Because the inner object accepts comparison and regex operators, `$elemMatch` extends blind extraction (see [Comparison operators](comparison-operators.md) and [Regex](regex.md)) to data held inside array-of-subdocument fields, one element condition per request. Combine with `$exists` to first confirm an element-level field is present (see [Exists](exists.md)).

## Tools

- **Burp Repeater**: craft $in, $nin, and $all arrays as JSON or repeated bracket keys.
- **nosqli**: scan parameters for array-operator injection.
- **NoSQLMap**: automate array-payload enumeration.

## References

- [MongoDB: $in](https://www.mongodb.com/docs/manual/reference/operator/query/in/)
- [MongoDB: $all](https://www.mongodb.com/docs/manual/reference/operator/query/all/)
- [MongoDB: $elemMatch](https://www.mongodb.com/docs/manual/reference/operator/query/elemMatch/)
- [PayloadsAllTheThings: NoSQL Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/NoSQL%20Injection)
