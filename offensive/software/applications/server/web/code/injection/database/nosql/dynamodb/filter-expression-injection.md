---
title: "DynamoDB filter expression injection: widening Query and Scan result sets"
description: "When user input shapes a FilterExpression or KeyConditionExpression string (or its ExpressionAttributeNames/Values), an attacker can broaden the predicate and return items outside the intended scope."
keywords:
  - filter expression injection
  - DynamoDB
  - FilterExpression
  - KeyConditionExpression
  - ExpressionAttributeValues
  - Query Scan
  - NoSQL injection
---

# Filter expression injection

`Query` and `Scan` narrow their results with a `FilterExpression`, and `Query` selects the partition/sort range with a `KeyConditionExpression`. Both are strings parsed by DynamoDB, and both reference attribute names through `#`-prefixed placeholders resolved from `ExpressionAttributeNames` and values through `:`-prefixed placeholders resolved from `ExpressionAttributeValues`. Injection occurs when attacker input is concatenated into the expression text itself, or when the attacker controls the **keys** of the attribute-name/value maps so that the placeholders resolve to attributes or operators the developer did not intend.

## Vulnerable patterns

String-built expression, the obvious sink. Expression strings do not take inline literals the way PartiQL does: values must be `:`-placeholders, so the injectable pattern concatenates a **clause or operator** (not a value) into the expression text:

```python
# account_id is bound safely with :aid, but extra_filter is attacker text joined in
expr = f"account_id = :aid AND {extra_filter}"
resp = table.scan(
    FilterExpression=expr,
    ExpressionAttributeValues={":aid": {"S": account_id}},
)
```

Whatever `extra_filter` contains becomes part of the parsed predicate.

Attribute-name/value maps driven by request structure, the subtler sink:

```python
# filters is a dict taken straight from JSON query params
names = {f"#{k}": k for k in filters}
values = {f":{k}": {"S": v} for k, v in filters.items()}
expr = " AND ".join(f"#{k} = :{k}" for k in filters)
table.scan(FilterExpression=expr,
           ExpressionAttributeNames=names,
           ExpressionAttributeValues=values)
```

Here the client chooses which attributes are compared and can drop the tenant guard entirely, or submit crafted keys.

## Exploitation

**Inject `OR` to defeat the guard.** There is no quoted-literal breakout (values are placeholders), so the primitive is appending a clause joined with `OR` to a function tautology. Injected into `extra_filter` above:

```
attribute_exists(account_id) OR attribute_exists(account_id)
```

makes the expression `account_id = :aid AND attribute_exists(account_id) OR attribute_exists(account_id)`. Since `AND` binds tighter than `OR`, this evaluates as `(account_id = :aid AND ...) OR attribute_exists(account_id)`, true for every item carrying that attribute, so the `Scan` returns the whole table.

**`attribute_exists` / `attribute_not_exists` as tautologies.** These functions are the expression-language equivalent of `1=1`. `attribute_exists(<partition key>)` is true for every item; `attribute_not_exists(<any always-present attribute>)` is reliably false and useful for negative tests and boolean inference.

**Widen a key condition.** On `Query`, the `KeyConditionExpression` controls which partition is read. Injecting a broader comparison or an extra `begins_with` on the sort key expands the returned range:

```
pk = :pk AND begins_with(sk, :empty)
```

with `:empty` set to `""` matches every sort key under the partition.

**Operator swap via the maps.** When only values are parameterized but the **operator** is chosen from input, flip an equality to an inequality or a `>` to widen matches:

```
#attr <> :v     # instead of  #attr = :v  → returns everything except one value
```

**Drop the tenant filter.** In the map-driven sink, simply omit the `account_id`/tenant key from the submitted filter object. If the backend builds the whole expression from the client's dict, nothing constrains the scan to the caller's data, and the response contains cross-tenant items.

**Blind inference.** Where the response only signals match-or-no-match, chain `attribute_exists`/`contains`/`begins_with` against guessed attribute names and value prefixes to confirm hidden fields and recover their contents a character at a time.

Note that `FilterExpression` is applied **after** items are read and counted, so a widened filter still consumes and returns everything the key condition selected, making it an effective path to bulk disclosure via repeated paginated `Scan`/`Query` calls.

## References

- [AWS DynamoDB: Filter expressions for Scan](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Scan.html#Scan.FilterExpression)
- [AWS DynamoDB: Condition and filter expression reference](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Expressions.OperatorsAndFunctions.html)
- [AWS DynamoDB: Expression attribute names and values](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Expressions.ExpressionAttributeNames.html)
