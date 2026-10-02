---
title: "OGNL injection: Object-Graph Navigation Language abuse in Struts2"
description: "Apache Struts2 evaluates OGNL from request data through %{...} and ${...}, and once the member-access sandbox is cleared the expression reaches Runtime and ProcessBuilder for code execution."
keywords:
  - OGNL injection
  - Struts2 OGNL
  - "%{} expression"
  - member access sandbox
  - OGNL RCE
---

# OGNL

OGNL (Object-Graph Navigation Language) is the expression language Struts2 uses to move values between the request, the value stack, and the view. Tag attributes, type conversion, and parameter names are all evaluated as OGNL, written `%{...}` in tags and reachable as `${...}` in some contexts. When request data flows into a string that Struts then evaluates, through a forced double evaluation, a crafted parameter name, or a tag attribute built from input, the OGNL grammar runs against the value stack.

OGNL is a full object language: it names classes, constructs objects, and calls methods. Modern Struts2 wraps evaluation in a sandbox, a `SecurityMemberAccess` with excluded classes and package names enforced through the OGNL context, so a bare `java.lang.Runtime` call is blocked. The exploitation pages below cover both proving evaluation without touching the sandbox and clearing the member-access restriction to reach the runtime.

- **[Directory listing](directory-listing.md)**: the blind arithmetic and string-evaluation probe that confirms OGNL evaluation before any runtime call.
- **[Remote code execution](remote-code-execution.md)**: restoring member access and reaching `Runtime` and `ProcessBuilder`.
- **[Remote file inclusion](remote-file-inclusion.md)**: constructing `java.io.File` and `java.net.URL` to read local files and fetch remote content.

## Tools

- **Burp Suite**: Repeater and Intruder for OGNL injection into Struts2 sinks.
- **J2EEScan**: Burp extension that detects OGNL and Struts expression injection.
- **struts-pwn**: PoC tool for Struts2 OGNL vectors.

## References

- [Apache Commons OGNL Language Guide](https://commons.apache.org/proper/commons-ognl/language-guide.html)
- [Apache Struts2 Security](https://struts.apache.org/security/)
- [PayloadsAllTheThings: Java OGNL Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Java)
