---
title: "NRPE abuse: the Nagios remote plugin executor"
description: "NRPE on TCP 5666 runs predefined check commands on a monitored host for the Nagios server. Where the agent allows command arguments (dont_blame_nrpe) and a command passes them to a shell, an attacker who reaches 5666 injects arguments and shell metacharacters to run arbitrary commands on the host, turning a monitoring agent into remote code execution."
keywords:
  - nrpe
  - 5666
  - dont_blame_nrpe
  - argument injection
  - command execution
---

# NRPE abuse

NRPE (Nagios Remote Plugin Executor) listens on TCP 5666 on monitored hosts and runs predefined check commands on behalf of the Nagios server, returning the result. By default it only runs the commands defined in its config and does not accept arguments, but two common relaxations open it up. If `dont_blame_nrpe=1` (allow arguments) is set, the server can pass arguments into command definitions that use `$ARG1$` etc., and if a command definition passes those arguments to a shell or a plugin that interprets them, an attacker who can reach 5666 injects shell metacharacters through the arguments to run arbitrary commands on the host. Even without argument passing, a poorly-defined command or a plugin that shells out on attacker-influenced input is exploitable. The result is command execution on the monitored host as the NRPE user.

```bash
# confirm the agent and list reachable behaviour
check_nrpe -H <target>                            # version; no command = agent present
check_nrpe -H <target> -c check_users             # run a defined command
# argument injection where dont_blame_nrpe=1 and a command passes args to a shell:
check_nrpe -H <target> -c <cmd_using_ARG> -a 'arg; id'         # ; command injection
check_nrpe -H <target> -c check_command -a '$(id)'            # if args reach a shell
# classic metachars that break out where the plugin/command invokes a shell: ; | ` $() &&
```

## Exploitation notes

- The precondition is argument passing (`dont_blame_nrpe=1`) plus a command whose definition hands the argument to a shell or an interpretive plugin; test defined commands with injected metacharacters in `-a` arguments.
- Reaching 5666 is often possible from within the network (agents are firewalled to the server, but flat networks and misconfigs expose them), and a compromised Nagios server can reach every agent; this is a strong pivot to all monitored hosts.
- Execution is as the NRPE service user on the monitored host; use it to read files, grab credentials, and move laterally.
- This is the agent-side command surface; injection into the server-side command definitions is [Command and plugin injection](command-and-plugin-injection.md).

## Tools

- [check_nrpe](https://github.com/NagiosEnterprises/nrpe)

## References

- [NRPE security (dont_blame_nrpe)](https://github.com/NagiosEnterprises/nrpe/blob/master/SECURITY.md)
- [HackTricks: NRPE](https://book.hacktricks.xyz/network-services-pentesting/5666-pentesting-nrpe)
