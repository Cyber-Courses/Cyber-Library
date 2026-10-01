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

> **Scope.** For authorized penetration tests, CTF labs, and code review of systems you own or are contracted to assess.

## Vulnerable patterns

Guard string built from input:

```python
# owner comes from the request
table.update_item(
    Key={"id": {"S": item_id}},
    UpdateExpression="SET balance = :b",
    ConditionExpression=f"owner = '{owner}'",
    ExpressionAttributeValues={":b": {"N": new_balance}},
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

**Neutralize the ownership guard with `OR`.** If `owner` is concatenated into the condition, close it and append a clause that is always true, so the write commits regardless of who owns the item:

```
anyone' OR attribute_exists(id) OR 'x'='x
```

The condition becomes `owner = 'anyone' OR attribute_exists(id) OR 'x'='x'`, which holds for every existing item, letting the attacker update records they do not own.

**Force a "create-if-absent" guard to pass and overwrite.** A `PutItem` protected by `attribute_not_exists(pk)` is meant to refuse clobbering an existing item. If the condition text is attacker-influenced, replace the guard with a tautology so the put overwrites the victim's item:

```
attribute_not_exists(pk) OR attribute_exists(pk)
```

Either branch now covers every case, so the write always lands and silently overwrites.

**Satisfy the optimistic-lock check.** In the operand-controlled pattern, the attacker supplies `expected_version`. Reading the current version first (via any disclosure path) and echoing it back makes `version = :v` pass, enabling a lost-update that stomps a concurrent writer's change. Where the value is coerced loosely, supplying a value that matches many items (or widening the operator to `>=`) broadens which items the write will touch.

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
