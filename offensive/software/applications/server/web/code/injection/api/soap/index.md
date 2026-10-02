---
title: "SOAP abuse: dispatch, headers, and body unmarshaling"
description: "SOAP routes XML envelopes by action and body to operation handlers. The attack surface is action-versus-body dispatch confusion, header processing such as WS-Security, and body parameter unmarshaling."
keywords:
  - SOAP injection
  - SOAPAction
  - WS-Security
  - SOAP body
  - XML dispatch
---

# SOAP

SOAP wraps each call in an XML envelope with a header and a body, and the server routes it to an operation handler based on an action hint and the body's first element. The layered structure, transport metadata, a `SOAPAction` header, WS-* headers, and a typed body, creates several independent places where routing and authorization can disagree with what the handler actually executes.

## The three surfaces

- **[SOAPAction spoofing](soapaction-spoofing.md)**: routing or authorization keyed on the `SOAPAction` header while the body names a different operation, so a request authorized for one action runs another on a shared endpoint.
- **[Header element injection](header-element-injection.md)**: WS-Addressing, WS-Security, and custom SOAP headers parsed from a partially trusted message, where replay, signature stripping, or `mustUnderstand` handling bugs live in the middleware.
- **[Body parameter injection](body-parameter-injection.md)**: body children and wrapped document parameters built from input, where type confusion, duplicate elements, and `xsi:nil` tricks change how the server unmarshals and authorizes the call.

## Why the envelope multiplies the surface

A REST call has one place that says what it is: the method and path. A SOAP call says it in several, the action header, the body's operation element, and the WS-* headers, and a framework may trust one while executing from another. The envelope is also parsed by a general XML stack, so the raw markup concerns from [XML processing](../../markup/xml-processing/index.md) apply underneath; the pages here focus on what is specific to SOAP operation binding rather than generic XML parsing. The test is to make the action header, the body operation, and the security headers disagree, and see which one the server believes.

## Tools

- **Burp Suite**: intercepting and tampering SOAP envelopes, headers, and the SOAPAction.
- **SoapUI**: generating requests from a WSDL and exercising operations.
- **Wsdler (Burp extension)**: parsing WSDL definitions into editable Burp requests.

## References

- [OWASP: Testing for SOAP](https://owasp.org/www-project-web-security-testing-guide/)
- [W3C: SOAP Version 1.2](https://www.w3.org/TR/soap12-part1/)
