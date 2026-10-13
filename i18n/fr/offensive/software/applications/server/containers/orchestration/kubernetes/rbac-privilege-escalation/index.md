---
order: 3
title: "Escalade de privilèges RBAC : transformer une identité faible en administrateur du cluster"
description: "Le RBAC Kubernetes accorde des verbes sur des ressources, et plusieurs combinaisons verbe-ressource permettent à une identité restreinte de s'octroyer davantage : des roles trop permissifs, les verbes escalate et bind, l'usurpation d'identité, la signature de certificats, l'API TokenRequest et la capacité de créer des pods. Chacune convertit un service account modeste en contrôle large ou total du cluster."
keywords:
  - rbac kubernetes
  - escalade de privilèges
  - escalate bind
  - impersonate
  - service account
---

# Escalade de privilèges RBAC

Kubernetes autorise les actions sous forme de verbes (`get`, `list`, `create`, `update`) sur des ressources (`pods`, `secrets`, `roles`). Le RBAC est censé empêcher une identité de s'octroyer plus qu'elle n'a, mais plusieurs permissions spécifiques brisent ce confinement. En détenir une seule transforme un service account restreint en une voie vers l'admin du cluster : un role trop large, les verbes `escalate` ou `bind` qui défont le contrôle anti-escalade, `impersonate` pour devenir une autre identité, la signature de certificats pour émettre un certificat client privilégié, l'API TokenRequest pour émettre les tokens d'autres service accounts, ou la capacité de créer des pods et d'en planifier un qui possède son nœud.

Cartographiez d'abord les permissions de l'identité :

```bash
kubectl auth can-i --list                           # the full rule set for this identity
# flag the dangerous ones:
kubectl auth can-i create pods --all-namespaces
kubectl auth can-i get secrets --all-namespaces
kubectl auth can-i impersonate users
kubectl auth can-i create clusterrolebindings
kubectl auth can-i '*' '*'
```

## Sous-sujets

- **[Roles trop permissifs](over-permissive-roles.md)** : roles à wildcard ou lisant les secrets.
- **[Verbes escalate et bind](escalate-and-bind-verbs.md)** : s'octroyer plus que l'on ne détient.
- **[Usurpation d'identité](impersonation.md)** : agir comme un autre utilisateur, groupe ou service account.
- **[Approbation de CSR](csr-approval.md)** : émettre un certificat client privilégié.
- **[API TokenRequest](tokenrequest-api.md)** : émettre des tokens pour d'autres service accounts.
- **[Abus de token de service account](service-account-token-abuse.md)** : monter ou réutiliser des tokens puissants.
- **[Création de pod vers le nœud](pod-creation-to-node.md)** : planifier un pod qui possède son nœud.

## Références

- [Kubernetes: RBAC authorization](https://kubernetes.io/docs/reference/access-authn-authz/rbac/)
- [Kubernetes: privilege escalation prevention](https://kubernetes.io/docs/reference/access-authn-authz/rbac/#privilege-escalation-prevention-and-bootstrapping)
- [HackTricks: Kubernetes RBAC](https://book.hacktricks.xyz/pentesting-cloud/kubernetes-security/kubernetes-role-based-access-control-rbac)
