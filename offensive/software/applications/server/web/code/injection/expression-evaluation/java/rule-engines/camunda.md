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

The exposure appears wherever expression text or a deployable model is assembled from untrusted input, for example a condition or delegate expression taken from user input, or a process variable interpolated into an evaluated expression:

```xml
<conditionExpression xsi:type="tFormalExpression">${userControlled}</conditionExpression>
```

Camunda also evaluates expressions supplied through its REST API (for example condition evaluation and variable expressions), widening where injected EL can enter.

## Reaching the runtime through JUEL

JUEL method resolution reaches `java.lang.Runtime`. An EL expression obtains the runtime and executes a command:

```
${Runtime.getRuntime().exec('id')}
```

Where `Runtime` is not directly resolvable in the EL context, pivot through an object's class to reach it:

```
${''.getClass().forName('java.lang.Runtime').getMethod('getRuntime').invoke(null).exec('id')}
```

Output is read back by wrapping the returned process stream in the same EL grammar.

## Script tasks are a more direct path

If the attacker controls a script task or an inline script expression, Camunda runs the named language directly through JSR-223, with Groovy and JavaScript commonly on the classpath. That is immediate execution without the EL method-chaining:

```groovy
"id".execute().text
```

A JavaScript (Nashorn/Graal) script task reaches `java.lang.Runtime` the same way. Because script tasks execute attacker code with no EL indirection, they are the cleaner sink wherever the process model or a script resource is controllable.

## Shell features need an argument vector

`Runtime.exec(String)` tokenizes on whitespace with no shell, so `$(...)`, pipes, and redirection stay literal. Build the vector for shell behavior, shown in EL:

```
${Runtime.getRuntime().exec(new String[]{'/bin/bash','-c','id > /tmp/o 2>&1'})}
```

On Windows use `new String[]{'cmd.exe','/c','whoami'}`. The Groovy `"...".execute()` form shells differently, so for pipes there build `["/bin/bash","-c","..."].execute()` with an explicit list.

## References

- [Camunda: expression language](https://docs.camunda.org/manual/latest/user-guide/process-engine/expression-language/)
- [Camunda: scripting](https://docs.camunda.org/manual/latest/user-guide/process-engine/scripting/)
- [Jakarta Expression Language specification](https://jakarta.ee/specifications/expression-language/)
