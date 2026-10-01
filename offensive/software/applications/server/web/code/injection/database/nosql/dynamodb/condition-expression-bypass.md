---
title: "DynamoDB condition expression bypass: forcing conditional writes to pass"
description: "Manipulating ConditionExpression operands so a conditional write guard passes when it should fail, enabling unauthorized create, update, delete, and overwrite of items."
keywords:
  - condition expression bypass
  - DynamoDB
  - ConditionExpression
  - attribute_not_exists
  - conditional write
  - optimistic locking
  - NoSQL injection
---

# Condition expression bypass

DynamoDB's conditional writes (`PutItem`, `UpdateItem`, `DeleteItem`, and the `ExecuteStatement` write forms) take a `ConditionExpression` that must evaluate true for the write to commit. Applications lean on these guards for correctness and authorization: `attribute_not_exists(pk)` to prevent overwriting an existing item, an equality check to enforce ownership, or a version compare for optimistic locking. When attacker input shapes the condition string or the `ExpressionAttributeNames`/`ExpressionAttributeValues` that feed it, the guard can be made to pass when it should fail, turning a protected write into an unauthorized create, update, delete, or overwrite.

## Vulnerable patterns

Guard string built from input. As with filter expressions, values must be `:`-placeholders, so the injectable pattern concatenates a **clause or operator**, not a quoted value:

```python
# guard is attacker text joined into the condition; :o holds the real owner value
cond = f"owner = :o AND {guard}"
table.update_item(
    Key={"id": {"S": item_id}},
    UpdateExpression="SET balance = :b",
    ConditionExpression=cond,
    ExpressionAttributeValues={":o": {"S": owner}, ":b": {"N": new_balance}},
)
```

Guard operand chosen from input (the subtler case):

```python
# expected_version supplied by the client
table.update_item(
    Key=key,
    UpdateExpression="SET #d = :d",
    ConditionExpression="version = :v",
    ExpressionAttributeValues={":v": {"N": client["expected_version"]},
                               ":d": {"S": client["data"]}},
)
```

## Exploitation

**Neutralize the ownership guard with `OR`.** With a clause concatenated into the condition, append an `OR` to a function tautology so the write commits regardless of owner. Injected into `guard` above:

```
attribute_exists(id) OR attribute_exists(id)
```

makes the condition `owner = :o AND attribute_exists(id) OR attribute_exists(id)`. Since `AND` binds tighter than `OR`, this is true for every existing item, letting the attacker update records they do not own.

**Force a "create-if-absent" guard to pass and overwrite.** A `PutItem` protected by `attribute_not_exists(pk)` is meant to refuse clobbering an existing item. If the condition text is attacker-influenced, replace the guard with a tautology so the put overwrites the victim's item:

```
attribute_not_exists(pk) OR attribute_exists(pk)
```

Either branch now covers every case, so the write always lands and silently overwrites.

**Not a bypass: the expected-version operand.** Supplying the `version` you previously read is the *intended* optimistic-lock protocol, not an injection. The key already identifies one item, DynamoDB does not loosely coerce the typed value, and a concurrent write changes the version so the condition then fails. It becomes a problem only if the application separately mistakes the version for authorization. The real bypass in the operand-controlled pattern is operator injection, next.

**Invert the comparison via operator injection.** When only the operand is parameterized but the comparison token comes from input, flip the guard's sense:

```
version <> :v      # was  version = :v  → passes for every item except the real one
```

combined with a key the attacker controls, this reliably commits writes the equality guard was meant to block.

**Delete past the guard.** The same primitives apply to `DeleteItem` with a `ConditionExpression`: a tautological condition removes authorization from a delete, so an attacker-chosen key can be destroyed even though the guard was intended to restrict deletion to the owner. (This is an authorization bypass on the write, not a trash-emptying or bulk-purge operation.)

**Transaction guards.** `TransactWriteItems` attaches a `ConditionCheck` and per-item `ConditionExpression`s; the same break-out and tautology techniques defeat an individual check and let the whole transaction commit, which is useful when a balance or inventory invariant is enforced only through that check.

Match attribute names and types to the target table; where the condition's outcome is observable only through success/failure of the write, use it as a boolean oracle to infer attribute presence and values before committing the final bypassing write.

## References

- [AWS DynamoDB: Condition expressions](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Expressions.ConditionExpressions.html)
- [AWS DynamoDB: Comparison operators and functions](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Expressions.OperatorsAndFunctions.html)
- [AWS DynamoDB: Working with items, conditional writes](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/WorkingWithItems.html#WorkingWithItems.ConditionalUpdate)
