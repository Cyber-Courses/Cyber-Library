---
title: "SOAPAction spoofing"
order: 1
description: "When routing or authorization is keyed on the SOAPAction header while execution reads the body's operation, a request authorized for one action runs a different one on a shared endpoint."
keywords:
  - SOAPAction spoofing
  - dispatch confusion
  - operation routing
  - SOAP authorization
  - shared endpoint
---

# SOAPAction spoofing

A SOAP request advertises its operation in two places: the `SOAPAction` HTTP header and the first child element of the SOAP Body. When a server, a gateway, or a security filter makes its routing or authorization decision from the header, but the handler executes the operation named in the body, the two can be made to disagree. A request that presents an innocuous action in the header and a sensitive operation in the body is authorized against the header and executed from the body.

## The mismatch

The header says one thing:

```
POST /services/Endpoint HTTP/1.1
SOAPAction: "urn:GetPublicStatus"
Content-Type: text/xml
```

The body says another:

```xml
<soap:Envelope xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/">
  <soap:Body>
    <DeleteUser xmlns="urn:AdminService">
      <id>104</id>
    </DeleteUser>
  </soap:Body>
</soap:Envelope>
```

A filter that allows `GetPublicStatus` and blocks `DeleteUser` inspects the header, sees the permitted action, and passes the request through. The dispatch layer then reads the Body, finds `DeleteUser`, and runs it. The access-control decision and the execution used different inputs.

## Where it applies

The flaw appears wherever action-based routing sits in front of body-based execution: a WS policy or XML gateway that matches on `SOAPAction`, an authorization module keyed on the action string, or a reverse proxy that routes by header while the application dispatches by body. It is especially common on endpoints that host many operations behind one URL, since the header is the cheap way to tell them apart and is trusted to match the body.

## Confirming the flaw

Send a request whose `SOAPAction` names a permitted, low-privilege operation and whose Body names a restricted one, and observe which executes. If the restricted operation runs, the server dispatches from the body while something upstream authorized from the header. Variations include an empty or omitted `SOAPAction` (some stacks then fall back to the body, others reject), and a header that names a nonexistent action to probe whether routing fails open to the body. The underlying fix is to require the action header and the body operation to match before dispatch, so a mismatch is the signal to look for.

## Tools

- **Burp Repeater**: setting a permitted SOAPAction while the body names a restricted operation.
- **SoapUI**: building per-operation requests from the WSDL to pair against mismatched actions.
- **Wsdler (Burp extension)**: enumerating operations from the WSDL to target.

## References

- [OWASP: Testing for SOAP](https://owasp.org/www-project-web-security-testing-guide/)
- [W3C: SOAP Version 1.2 Part 2 (SOAPAction)](https://www.w3.org/TR/soap12-part2/)
