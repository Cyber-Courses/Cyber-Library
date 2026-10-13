---
title: "OpenVPN: configuration, credential, and key-handling attacks"
order: 2
description: "OpenVPN's cryptography is sound, so attacks target its configuration and key handling: stolen client profiles and embedded keys, a missing or weak tls-auth/tls-crypt HMAC that exposes the control channel, username/password auth sprayed against the server, and reused or poorly protected client certificates and static keys."
keywords:
  - openvpn
  - tls-auth
  - client profile
  - static key
  - credential
---

# OpenVPN

OpenVPN is cryptographically well-regarded, so attacking it means attacking how it is deployed rather than the protocol. The richest target is the client profile (`.ovpn`): it often embeds the CA, client certificate, private key, and sometimes credentials, so a stolen profile authenticates the attacker directly. The optional `tls-auth`/`tls-crypt` HMAC protects the control channel; when it is absent or its key is leaked (it too lives in the profile), the control channel is exposed to probing and some attacks. Where OpenVPN uses username/password auth, the server is a spraying target. And static-key or certificate reuse across clients means one recovered key opens the VPN.

```bash
# steal/examine a client profile for embedded keys and creds
grep -iE '<cert>|<key>|<tls-auth>|auth-user-pass' client.ovpn
openssl rsa -in client.key -noout -text 2>/dev/null     # confirm a usable private key
# reuse the profile to connect as that client
openvpn --config client.ovpn
# where username/password auth is used, spray the server (see password brute force)
```

## Exploitation notes

- Client profiles are the prize: hunt for `.ovpn` files (and `.key`/`.crt`) on endpoints, backups, images, and repositories; an embedded private key and certificate authenticate as that client with no further credential.
- A missing `tls-auth`/`tls-crypt`, or a leaked HMAC key (recoverable from a profile), removes the control-channel protection that otherwise blocks unauthenticated probing of the OpenVPN server.
- Shared/static keys and reused client certificates mean one recovered credential opens the VPN for the attacker; check whether the deployment issues per-client certs or reuses one.
- Username/password OpenVPN is sprayable like any portal ([password brute force](../authentication/password-brute-force.md)); certificate reuse and theft follow [certificate abuse](../authentication/certificate-abuse.md).

## References

- [OpenVPN security and tls-auth](https://openvpn.net/community-resources/hardening-openvpn-security/)
- [HackTricks: OpenVPN](https://book.hacktricks.xyz/network-services-pentesting/openvpn)
