---
order: 2
title: "Abus des métadonnées gRPC"
description: "Lorsqu'un serveur gRPC fait confiance aux métadonnées par appel, comme l'authority ou des en-têtes personnalisés, pour l'autorisation, le routage ou la sélection de tenant, un client qui définit ces valeurs librement usurpe une identité ou atteint des endpoints internes."
keywords:
  - métadonnées gRPC
  - grpc-metadata
  - en-tête authority
  - usurpation de tenant
  - autorisation par métadonnées
---

# Abus des métadonnées

Chaque appel gRPC porte une carte de métadonnées : des paires clé-valeur envoyées avec le message, y compris des entrées réservées comme `:authority` et des clés personnalisées arbitraires. Les métadonnées sont un canal auxiliaire pratique pour les préoccupations transversales, de sorte que le code applicatif les lit souvent pour l'autorisation, la sélection de tenant ou le routage. Le hic est que le client les contrôle complètement, de sorte que toute décision prise à partir de métadonnées non signées fait confiance à une entrée de l'attaquant qui ressemble simplement à de l'infrastructure.

## Revendications de confiance dans les métadonnées

Un schéma courant place l'identité ou le rôle dans une clé de métadonnées personnalisée définie par une gateway, puis la lit dans un intercepteur en aval. Si ce service est joignable directement, ou si la gateway ne supprime pas les copies fournies par le client, le client définit la clé lui-même :

```
grpcurl -plaintext \
  -H 'x-user-id: 1' -H 'x-user-role: admin' \
  -d '{}' target:50051 billing.BillingService.ListAllInvoices
```

Le serveur lit `x-user-role` et autorise en tant qu'admin, car il a traité une convention interne comme si les clients ne pouvaient pas la forger. Il en va de même pour les clés de tenant (`x-tenant-id`), où écraser la valeur lit ou écrit les données d'un autre tenant.

## Authority et routage

Le pseudo-en-tête `:authority` et les métadonnées de type hôte sont parfois utilisés pour choisir un backend, un service virtuel ou un shard de tenant. Le définir à un nom interne peut router un appel vers un service que le client ne devrait pas atteindre, ou sélectionner un contexte de tenant privilégié :

```
grpcurl -authority 'internal-admin.svc.cluster.local' -plaintext \
  -d '{}' target:50051 admin.AdminService.Health
```

## Pourquoi on lui fait confiance alors qu'il ne le faudrait pas

Les métadonnées ressemblent à de la tuyauterie de transport, de sorte que les développeurs les traitent comme dignes de confiance comme ils le feraient d'une variable d'environnement définie par le serveur. Mais sur le réseau, ce n'est qu'une entrée client sous un autre nom. Une gateway qui injecte des revendications vérifiées doit aussi supprimer toute copie des mêmes clés fournie par le client, et les services en aval doivent distinguer les métadonnées qu'ils ont signées de celles qu'un appelant a fournies. Le test consiste à ajouter ou écraser les métadonnées d'autorisation, de tenant et d'authority sur un appel direct, et à voir si le serveur agit dessus ; si un intercepteur lit une clé sans vérifier son origine, l'appel est autorisé sur une entrée forgée.

## Outils

- **grpcurl** : définition de métadonnées personnalisées et de l'authority avec -H et -authority sur un appel direct.
- **grpcui** : ajout ou écrasement de métadonnées via un client interactif.
- **Burp Repeater** : altération des en-têtes Grpc-Metadata- mappés derrière grpc-gateway ou grpc-web.

## Références

- [gRPC: Metadata](https://grpc.io/docs/guides/metadata/)
- [OWASP API Security Top 10: Broken Authentication](https://owasp.org/API-Security/editions/2023/en/0xa2-broken-authentication/)
