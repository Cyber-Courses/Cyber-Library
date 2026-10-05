---
title: "Weak TLS on 2376: reaching the Docker daemon past weak certificate checks"
description: "Reaching a Docker daemon on the TLS port 2376 when mutual TLS is misconfigured, such as a daemon that does not verify client certificates or accepts a leaked or widely shared certificate, giving the same root-equivalent control as the plaintext port."
keywords:
  - docker 2376
  - docker TLS
  - mutual TLS
  - client certificate
  - exposed daemon
---

# Weak TLS on 2376

Port `2376` is meant to be mutual TLS: the daemon presents a certificate and verifies the client's. It is only as strong as that verification. A daemon started with TLS enabled but client verification off, or with a client certificate that has leaked or is shared across a fleet, is reachable by an attacker who holds or bypasses the certificate.

```bash
# No client-cert verification: plain TLS connects and controls the daemon
curl -sk https://<host>:2376/version
docker --tls -H tcp://<host>:2376 ps

# With a leaked client cert/key
docker --tlsverify --tlscert=client.pem --tlskey=key.pem -H tcp://<host>:2376 ps
```

## Exploitation notes

- The common failure is `--tls` without `--tlsverify`, which encrypts but does not authenticate the client.
- Client certificates and keys are often found in CI secrets, home directories, and images; treat a recovered `cert.pem`/`key.pem` as daemon access.
- Once connected, proceed as in [Host takeover via privileged run](host-takeover-via-privileged-run.md).

## References

- [Docker: protect the daemon socket with TLS](https://docs.docker.com/engine/security/protect-access/)
- [Docker Engine API](https://docs.docker.com/engine/api/)
