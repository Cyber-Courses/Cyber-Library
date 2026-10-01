---
title: "Camunda injection: code execution through JUEL expressions and script tasks"
description: "Camunda BPMN evaluates Unified EL expressions bound to Java methods and runs script-task code, so untrusted input into a process expression invokes methods and reaches the JVM runtime."
keywords:
  - Camunda
  - Camunda injection
  - JUEL
  - BPMN expression
  - script task
---

# Camunda

Camunda is a BPMN process and decision engine. Process models evaluate expressions at many points: sequence-flow conditions, service-task delegate expressions, input and output mappings, listeners, and DMN decision tables. The default expression language is Unified EL (JUEL), `${...}`, and Camunda binds EL resolution to the process variables and to Spring or CDI beans, meaning an expression can invoke Java methods. When a process definition, a form value, or a variable that feeds an expression is attacker-influenced, the injected EL runs in the engine and reaches the runtime through method resolution.

## Vulnerable pattern

The exposure needs attacker control of the expression **text**, not merely of a variable the expression reads. A process variable interpolated with `${someVar}` returns the variable's value as data; that value is not recursively parsed as EL, so controlling the variable alone is not injection. The real entry points are where untrusted text becomes the expression itself:

```xml
<!-- untrusted input concatenated INTO the condition expression -->
<conditionExpression xsi:type="tFormalExpression">${amount > USERINPUT}</conditionExpression>
```

or a deployable model the attacker can author, or an expression string handed to Camunda's REST API (the engine evaluates condition and expression fields it is given directly), which is the cleanest way in because the attacker supplies the EL text verbatim.

## Reaching the runtime through JUEL

JUEL (standard Unified EL) has no `import`, no `new`, no array literals, and no class-literal syntax, so it cannot name `java.lang.Runtime` directly unless the application exposed a bean by that name. The portable path reaches the class through reflection from a string literal, then invokes the static `getRuntime`:

```
${''.getClass().forName('java.lang.Runtime').getMethod('getRuntime').invoke(null).exec('id')}
```

Output is read back by wrapping the returned process stream through further reflective calls in the same EL grammar.

## Script tasks are a more direct path

If the attacker controls a script task or an inline script expression, Camunda runs the named language directly through JSR-223, with Groovy and JavaScript commonly on the classpath. That is immediate execution without the EL method-chaining:

```groovy
"id".execute().text
```

A JavaScript (Nashorn/Graal) script task reaches `java.lang.Runtime` the same way. Because script tasks execute attacker code with no EL indirection, they are the cleaner sink wherever the process model or a script resource is controllable.

## Shell features need a script task

`Runtime.exec(String)` tokenizes on whitespace with no shell, so `$(...)`, pipes, and redirection stay literal, and the reflective EL path above runs a single binary. JUEL cannot build a `String[]` argument vector (it has no `new` and no array literal), so shaping a shell invocation through pure EL is impractical. Where shell features are needed, the script-task path is the route: a Groovy script builds an explicit list and runs it through a shell.

```groovy
["/bin/bash","-c","id | base64 > /tmp/o"].execute()
```

On Windows the list is `["cmd.exe","/c","whoami"]`.

## References

- [Camunda: expression language](https://docs.camunda.org/manual/latest/user-guide/process-engine/expression-language/)
- [Camunda: scripting](https://docs.camunda.org/manual/latest/user-guide/process-engine/scripting/)
- [Jakarta Expression Language specification](https://jakarta.ee/specifications/expression-language/)
