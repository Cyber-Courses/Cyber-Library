---
order: 6
title: "Persistance : conserver l'accès à un cluster compromis"
description: "Après avoir pris le contrôle, un attaquant le conserve en plantant des mécanismes qui survivent à la rotation des identifiants et aux redémarrages de pods : des bindings RBAC accordant une identité durable, des charges de travail malveillantes et des cronjobs qui rétablissent l'accès, des static pods que le kubelet exécute directement, et des admission webhooks qui injectent des backdoors ou forgent des identifiants à chaque requête d'API pertinente."
keywords:
  - persistance kubernetes
  - backdoor rbac
  - static pod
  - admission webhook
  - cronjob
---

# Persistance

Conserver un cluster Kubernetes signifie survivre aux réponses évidentes : un token en rotation, un pod supprimé, un nœud corrigé. Les mécanismes durables se plantent dans l'état du cluster ou sur les nœuds pour leur survivre. Un binding RBAC accorde une identité à longue durée de vie qu'aucune rotation de token ne retire ; une charge de travail ou un cronjob recrée l'accès sur une planification ; un static pod s'exécute depuis le répertoire de manifestes d'un nœud sans objet d'API à supprimer ; et un admission webhook se place dans le chemin des requêtes, injectant des backdoors ou forgeant des identifiants à chaque création d'un objet correspondant.

```bash
# what can the current identity create for persistence?
kubectl auth can-i create clusterrolebindings
kubectl auth can-i create mutatingwebhookconfigurations
kubectl auth can-i create cronjobs -A
```

## Sous-thèmes

- **[Backdoor RBAC](rbac-backdoor.md)** : une identité durable via des bindings.
- **[Charges de travail malveillantes](malicious-workloads.md)** : des deployments et daemonsets qui rétablissent l'accès.
- **[CronJobs](cronjobs.md)** : ré-entrée planifiée.
- **[Static pods](static-pods.md)** : des pods que le kubelet exécute depuis un répertoire de manifestes d'un nœud.
- **[Admission webhooks](admission-webhooks.md)** : intercepter le chemin des requêtes de l'API.
- **[Backdoor par webhook de mutation](mutating-webhook-backdoor.md)** : injecter dans chaque objet correspondant.

## Références

- [Kubernetes: RBAC](https://kubernetes.io/docs/reference/access-authn-authz/rbac/)
- [MITRE ATT&CK: persistence (containers)](https://attack.mitre.org/tactics/TA0003/)
- [Microsoft: threat matrix for Kubernetes](https://www.microsoft.com/en-us/security/blog/2021/03/23/secure-containerized-environments-with-updated-threat-matrix-for-kubernetes/)
