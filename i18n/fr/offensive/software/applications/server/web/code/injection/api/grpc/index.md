---
order: 2
title: "Abus de gRPC : réflexion, métadonnées et typage dynamique"
description: "gRPC dispatche des messages protobuf vers des méthodes de service. La surface d'attaque est la réflexion du serveur qui expose la carte des RPC, les métadonnées auxquelles on fait confiance pour l'autorisation, et le typage dynamique de messages qui décode des types choisis par l'attaquant."
keywords:
  - injection gRPC
  - réflexion de serveur
  - métadonnées gRPC
  - protobuf Any
  - grpcurl
---

# gRPC

gRPC transporte des messages protobuf sur HTTP/2 vers des méthodes de service fortement typées. Le cadrage binaire et les stubs générés le font paraître fermé, mais les mêmes propriétés qui le rendent efficace, une carte de service découvrable, un canal auxiliaire de métadonnées et des types de messages dynamiques, sont là où le code applicatif accorde trop de confiance à l'appelant.

## Les trois surfaces

- **[Abus de la réflexion de serveur](server-reflection-abuse.md)** : le service de réflexion livrant la liste complète des services, méthodes et descripteurs de messages, transformant une API binaire opaque en une API navigable avec un outil comme `grpcurl`.
- **[Abus des métadonnées](metadata-abuse.md)** : la carte de métadonnées par appel (`:authority`, en-têtes personnalisés et, derrière grpc-gateway/grpc-web, en-têtes HTTP mappés via le préfixe `Grpc-Metadata-`) à laquelle on fait confiance pour l'autorisation, le routage ou la sélection de tenant, même si le client la définit librement.
- **[Confusion de messages dynamiques](dynamic-message-confusion.md)** : `google.protobuf.Any`, les registres de types et la gestion de `oneof` où le serveur décode un type choisi par l'attaquant dans un handler non sûr, un confused deputy entre types de messages.

## Pourquoi binaire ne signifie pas sûr

Le format filaire protobuf et le client généré suggèrent un contrat scellé, mais rien dans le cadrage binaire n'impose l'autorisation ni ne restreint les méthodes qu'un appelant peut invoquer. La réflexion annonce la surface, les métadonnées sont une entrée contrôlée par l'attaquant qui ressemble à de l'infrastructure, et le typage dynamique réintroduit exactement le problème « décoder des octets non fiables dans un type choisi » que le typage fort était censé supprimer. Le test pour chacun est le même : supposer que l'appelant peut nommer n'importe quelle méthode, définir n'importe quelle métadonnée et fournir n'importe quelle type URL, puis trouver où le serveur agit dessus sans vérifier.

## Outils

- **grpcurl** : liste des services et invocation des méthodes via gRPC.
- **grpcui** : client navigateur interactif pour un serveur gRPC.
- **blackboxprotobuf (extension Burp)** : décodage et altération des messages protobuf dans Burp.

## Références

- [gRPC: Server Reflection](https://grpc.io/docs/guides/reflection/)
- [OWASP API Security Top 10](https://owasp.org/API-Security/editions/2023/en/0x11-t10/)
