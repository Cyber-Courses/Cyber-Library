---
title: "Certificate validation bypass: abusing certificate-based SSH trust"
description: "Certificate-based SSH replaces per-host known_hosts pinning with a CA that signs host and user certificates. Weaknesses, a compromised or over-scoped CA key, missing principal and validity checks, accepting self-signed or wrong-CA certificates, let an attacker present a trusted host certificate for machine-in-the-middle or forge user certificates for authentication."
keywords:
  - ssh certificate
  - ca key
  - principals
  - certificate authority
  - trust
---

# Certificate validation bypass

Larger SSH deployments replace per-host `known_hosts` management with certificates: a certificate authority signs host certificates (so clients trust any host whose key the CA signed) and user certificates (so servers accept any user whose key the CA signed, within stated principals and validity). This scales trust but concentrates it in the CA, and weaknesses in the model or its validation are the attack surface. A compromised or over-scoped CA signing key lets an attacker mint trusted host certificates (for MITM) or user certificates (for authentication); and clients or servers that fail to check principals, validity windows, or the correct CA accept certificates they should reject.

```bash
# inspect a certificate's scope and validity
ssh-keygen -L -f id_ed25519-cert.pub           # principals, validity, CA, options
# with a compromised CA signing key, mint a host cert for MITM or a user cert for auth
ssh-keygen -s ca_key -I attacker -h -n <target-hostname> host_key.pub          # host cert
ssh-keygen -s ca_key -I attacker -n root -V +1d user_key.pub                   # user cert as root
ssh -i user_key -o CertificateFile=user_key-cert.pub root@<target>
```

## Exploitation notes

- The CA signing key is the crown jewel: possession lets you forge both host certificates (present a trusted key for [SSH MITM](ssh-mitm.md)) and user certificates (authenticate as any principal the server accepts), so hunt for the CA key as you would any high-value private key.
- Validation gaps enable attacks without the CA key: a server not enforcing the `principals` list, ignoring the validity window, or trusting the wrong CA accepts certificates it should not; test with a certificate that is out of principal or expired.
- User certificates forged for a privileged principal (`root`, an admin account) are direct authentication, bounded only by what the server's `TrustedUserCAKeys`/principals allow.
- `ssh-keygen -L` reveals a certificate's principals, validity, and options, the facts that determine whether a given certificate is accepted where.

## References

- [OpenSSH certificate authentication (ssh-keygen -s)](https://man.openbsd.org/ssh-keygen#CERTIFICATES)
- [Facebook/Uber SSH CA design writeups](https://engineering.fb.com/2016/09/12/security/scalable-and-secure-access-with-ssh/)
