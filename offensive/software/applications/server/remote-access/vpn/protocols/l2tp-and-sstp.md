---
title: "L2TP and SSTP: inherited IPsec-PSK and TLS weaknesses"
order: 5
description: "L2TP provides no tunnel protection itself and is commonly paired with IPsec. Its assessment depends on the actual IKE mode and authentication method; offline PSK extraction is not a generic property of every L2TP/IPsec deployment. SSTP inherits TLS and certificate-validation requirements."
keywords:
  - l2tp
  - sstp
  - ipsec psk
  - tls
  - ppp
---

# L2TP and SSTP

L2TP and SSTP carry PPP through different protective layers. L2TP does not provide tunnel confidentiality by itself and is commonly deployed as L2TP/IPsec. Assess the IKE version, mode, authentication method, and transforms rather than assuming the pre-shared key is exposed: an offline PSK artifact can be collected from IKEv1 Aggressive Mode when PSK authentication is enabled, but that workflow does not apply automatically to Main Mode, IKEv2, or certificate-authenticated deployments. SSTP carries PPP over TLS on TCP 443. A practical interception finding depends on the negotiated TLS configuration and whether the client validates the server certificate; a self-signed certificate is not itself exploitable when the client explicitly trusts the correct certificate.

```bash
# L2TP/IPsec: first enumerate IKE; capture a PSK artifact only if Aggressive Mode is offered
ike-scan -M <target>
ike-scan -A -M -P psk.hash <target>; psk-crack -d wordlist.txt psk.hash
# SSTP (TLS on 443): grade the TLS and check certificate validation
nmap -p443 --script ssl-enum-ciphers <target>
openssl s_client -connect <target>:443 | openssl x509 -noout -issuer -subject
# inner PPP/MS-CHAPv2 (both protocols) is crackable if captured (see PPTP)
```

## Exploitation notes

- L2TP/IPsec inherits IPsec's security properties. Report a weak or shared PSK separately from an offline-cracking path, which requires evidence such as IKEv1 Aggressive Mode support; see [IPsec IKE](ipsec-ike.md) and [weak pre-shared keys](../authentication/weak-pre-shared-keys.md).
- SSTP inherits TLS's security properties. An unverified certificate enables interception, while a self-signed certificate that is correctly pinned or explicitly trusted does not by itself establish a vulnerability.
- Both can carry MS-CHAPv2 inside PPP. Capturing that handshake requires visibility inside the protected carrier, such as control of an endpoint or successful compromise of the outer IPsec/TLS layer; a passive observer sees only the outer tunnel.
- The inherited nature means these are rarely novel: attack the carrier (IPsec PSK or TLS), then the inner PPP credentials.

## References

- [RFC 2661: L2TP](https://www.rfc-editor.org/rfc/rfc2661.html)
- [RFC 3193: securing L2TP with IPsec](https://www.rfc-editor.org/rfc/rfc3193.html)
- [Microsoft SSTP overview](https://learn.microsoft.com/en-us/windows-server/remote/remote-access/)
