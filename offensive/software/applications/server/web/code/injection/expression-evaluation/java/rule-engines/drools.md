---
title: "Drools injection: code execution through the MVEL dialect"
description: "Drools rule conditions and consequences evaluate in the MVEL dialect, which has full Java access, so untrusted input built into a Drools expression or consequence runs MVEL and reaches the JVM runtime."
keywords:
  - Drools
  - Drools injection
  - MVEL dialect
  - DRL
  - JVM code execution
---

# Drools

Drools is a business-rules engine. Rules are written in DRL (Drools Rule Language), where each rule has a left-hand side of conditions over the working-memory facts and a right-hand side consequence that runs when the rule fires. The expressions in conditions, and the code in consequences, are evaluated by a dialect, and Drools' default expression dialect is MVEL. MVEL is a full JVM expression language with object construction, method invocation, and reflection, so the injection vector here is MVEL under Drools: anything that builds DRL text or an MVEL expression from untrusted input runs attacker MVEL, and MVEL reaches the runtime.

## Vulnerable pattern

The exposure appears wherever rule text is assembled dynamically rather than loaded from fixed resources, for example building DRL from a user-supplied condition or threshold and compiling it:

```java
String drl =
  "rule \"dynamic\"\n" +
  "when\n" +
  "  $o : Order( " + userCondition + " )\n" +
  "then\n" +
  "  " + userAction + "\n" +
  "end";
// drl compiled into a KieBase and fired
```

Both the woven condition and the consequence are MVEL. Applications that let users author rules, or that pass user input into an MVEL `eval` or a decision-table cell, expose the same dialect.

## Reaching the runtime

MVEL resolves class names and calls methods, so a consequence (or an injected MVEL expression) obtains the runtime and executes a command:

```
Runtime.getRuntime().exec("id");
```

MVEL also supports explicit construction, so `ProcessBuilder` works directly:

```
new java.lang.ProcessBuilder("id").start();
```

To capture output, wrap the process stream in the consequence, constructing a `java.util.Scanner` over `getInputStream()` and reading a token, then bind it back to a fact field or an inserted object that the application later reads.

## Shell features need an argument vector

`Runtime.exec(String)` tokenizes on whitespace and runs with no shell, so `$(...)`, pipes, and redirection stay literal. For shell behavior build the argument vector and let bash interpret it:

```
new java.lang.ProcessBuilder(new String[]{"/bin/bash","-c","id > /tmp/o 2>&1"}).start();
```

On Windows use `new String[]{"cmd.exe","/c","whoami"}`. If the application pinned the dialect to `java` instead of `mvel`, the consequence is compiled Java rather than MVEL but reaches the same `Runtime`/`ProcessBuilder` API, so the command sink is unchanged.

## Tools

- **Burp Suite**: Repeater to deliver MVEL condition and consequence payloads.
- Manual MVEL payloads reaching Runtime and ProcessBuilder.

## References

- [Drools documentation](https://docs.drools.org/)
- [MVEL language guide](http://mvel.documentnode.com/)
- [Drools: rule language reference (DRL)](https://docs.drools.org/latest/drools-docs/html_single/#drl-rules-con_drl-rules)
