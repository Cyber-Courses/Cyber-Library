---
title: "Certificate trust abuse: compromised and over-broad SSH CAs"
description: "Certificate-based SSH can replace per-host known_hosts pinning with a host CA and authorize users through a user CA. Compromise or over-broad trust of a signing key lets an attacker forge certificates only in the roles and scopes where that CA is trusted."
keywords:
  - ssh certificate
  - ca key
  - principals
  - certificate authority
  - trust
---

# Certificate trust abuse

Larger SSH deployments can use certificates in two distinct trust roles. Clients trust configured host CAs to authenticate servers, while servers trust configured user CAs—such as those listed by `TrustedUserCAKeys`—to authenticate users within the permitted principals and validity period. This scales trust but concentrates risk in each CA. A compromised or over-broad host CA can enable host impersonation where clients trust it; a compromised or over-broad user CA can enable authentication where servers trust it. One signing key enables both attacks only when its public key has deliberately been trusted in both roles.

```bash
# inspect a certificate's scope and validity
ssh-keygen -L -f id_ed25519-cert.pub           # principals, validity, CA, options
# with the relevant compromised CA key, mint a cert for the role where that CA is trusted
ssh-keygen -s ca_key -I attacker -h -n <target-hostname> host_key.pub          # host cert
ssh-keygen -s ca_key -I attacker -n root -V +1d user_key.pub                   # user cert as root
ssh -i user_key -o CertificateFile=user_key-cert.pub root@<target>
```

## Exploitation notes

- CA signing keys are high-value secrets, but their impact follows their configured role: a host CA trusted by clients can sign host certificates for [SSH MITM](ssh-mitm.md), and a user CA trusted by servers can sign user certificates for authentication.
- Over-broad CA distribution or principal policy expands that impact. Assess which clients trust each host CA and which servers trust each user CA instead of assuming a signing key is accepted everywhere or for both certificate types.
- A user certificate for a privileged principal (`root`, an admin account) succeeds only when the target server trusts its signing CA and the principal satisfies the effective `TrustedUserCAKeys`, `AuthorizedPrincipalsFile`, or authorized-key policy.
- `ssh-keygen -L` reveals a certificate's principals, validity, and options, the facts that determine whether a given certificate is accepted where.

## References

- [OpenSSH certificate authentication (ssh-keygen -s)](https://man.openbsd.org/ssh-keygen#CERTIFICATES)
- [Facebook/Uber SSH CA design writeups](https://engineering.fb.com/2016/09/12/security/scalable-and-secure-access-with-ssh/)
