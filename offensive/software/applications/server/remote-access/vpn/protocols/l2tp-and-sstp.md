---
title: "L2TP and SSTP: inherited IPsec-PSK and TLS weaknesses"
description: "L2TP provides no encryption itself and is paired with IPsec, so it inherits the IPsec pre-shared-key weaknesses, a weak or default L2TP/IPsec PSK is captured and cracked like any IKE PSK. SSTP tunnels PPP over TLS, so it inherits TLS configuration weaknesses: weak ciphers, downgrade, and certificate-validation gaps enabling interception."
keywords:
  - l2tp
  - sstp
  - ipsec psk
  - tls
  - ppp
---

# L2TP and SSTP

L2TP and SSTP are both carried by something else, and they inherit that carrier's weaknesses. L2TP provides no confidentiality on its own and is almost always deployed as L2TP/IPsec, where IPsec supplies the encryption using a pre-shared key; that PSK is frequently weak, default, or shared widely, and is captured and cracked exactly as any IKE PSK, after which the user credentials (PPP/MS-CHAP inside) are the next target. SSTP tunnels PPP over a TLS connection (TCP 443), so its security is TLS's: weak cipher suites, protocol downgrade, and especially unverified or self-signed certificates let a positioned attacker intercept, and the inner PPP authentication (often MS-CHAPv2) is then exposed.

```bash
# L2TP/IPsec: attack the IPsec PSK first (aggressive mode, if offered)
ike-scan -A -M -P psk.hash <target>; psk-crack -d wordlist.txt psk.hash
# SSTP (TLS on 443): grade the TLS and check certificate validation
nmap -p443 --script ssl-enum-ciphers <target>
openssl s_client -connect <target>:443 | openssl x509 -noout -issuer -subject
# inner PPP/MS-CHAPv2 (both protocols) is crackable if captured (see PPTP)
```

## Exploitation notes

- L2TP's security is entirely IPsec's: a weak/default L2TP/IPsec PSK is the same attack as [IPsec IKE](ipsec-ike.md) and [weak pre-shared keys](../authentication/weak-pre-shared-keys.md); a single shared PSK across all users is common and is the foothold.
- SSTP's security is entirely TLS's: unverified/self-signed certificates and weak ciphers enable MITM and interception, so assess it like any [weak-TLS](../weak-cryptography/weak-ciphers.md) endpoint.
- Both commonly carry MS-CHAPv2 inside the PPP layer, which is crackable from a captured handshake like [PPTP](pptp.md); so even after the outer layer, the inner authentication is attackable.
- The inherited nature means these are rarely novel: attack the carrier (IPsec PSK or TLS), then the inner PPP credentials.

## References

- [RFC 3931 (L2TPv3) and L2TP/IPsec](https://datatracker.ietf.org/doc/html/rfc3931)
- [Microsoft SSTP overview](https://learn.microsoft.com/en-us/windows-server/remote/remote-access/)
