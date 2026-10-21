---
title: "OpenVPN: configuration, credential, and key-handling attacks"
order: 2
description: "OpenVPN attacks commonly target configuration and key handling: exposed profiles, private keys, control-channel keys, and user credentials. Whether a recovered profile is sufficient depends on private-key encryption and the server's certificate, password, token, or multi-factor requirements."
keywords:
  - openvpn
  - tls-auth
  - client profile
  - static key
  - credential
---

# OpenVPN

OpenVPN attacks commonly target how it is deployed and how credentials are stored. A client profile (`.ovpn`) may embed or reference the CA, client certificate, private key, control-channel key, and an `auth-user-pass` source. A recovered profile is sufficient only when all authentication material it requires is present and usable. The private key may be passphrase-protected, and servers can require username/password, token, or multi-factor authentication in addition to a client certificate. The optional `tls-auth` or `tls-crypt` key adds control-channel protection; recovering it removes that additional gate but does not replace the VPN's certificate or user authentication.

```bash
# steal/examine a client profile for embedded keys and creds
grep -iE '<cert>|<key>|<tls-auth>|<tls-crypt>|auth-user-pass' client.ovpn
openssl pkey -in client.key -check -noout               # supports RSA and EC; may request a passphrase
# reuse the profile to connect as that client
openvpn --config client.ovpn
# where username/password auth is used, spray the server (see password brute force)
```

## Exploitation notes

- Treat a profile as an inventory of requirements: determine whether its key is embedded or external and encrypted or usable, and whether `auth-user-pass`, a token, or MFA is additionally required.
- A missing or recovered `tls-auth`/`tls-crypt` key removes additional control-channel protection, but it does not by itself authenticate a VPN session.
- Shared static keys and reused client certificates broaden the impact of recovered material, but successful access still depends on every authentication factor configured by the server.
- Username/password OpenVPN is sprayable like any portal ([password brute force](../authentication/password-brute-force.md)); certificate reuse and theft follow [certificate abuse](../authentication/certificate-abuse.md).

## References

- [OpenVPN security and tls-auth](https://openvpn.net/community-resources/hardening-openvpn-security/)
- [HackTricks: OpenVPN](https://book.hacktricks.xyz/network-services-pentesting/openvpn)
