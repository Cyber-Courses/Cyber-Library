---
order: 1
title: "Abus de la réflexion de serveur gRPC"
description: "Lorsqu'un serveur gRPC expose le service de réflexion dans un environnement où il ne le devrait pas, un attaquant énumère chaque service, méthode et descripteur de message, transformant une API binaire opaque en une API navigable."
keywords:
  - réflexion gRPC
  - grpcurl
  - ServerReflection
  - découverte d'API
  - descripteurs protobuf
---

# Abus de la réflexion de serveur

La réflexion de serveur gRPC est un service intégré qui répond, à l'exécution, avec l'ensemble complet des services, méthodes et descripteurs de messages du serveur. Il existe pour que les outils puissent appeler une API sans avoir les fichiers `.proto` sous la main. Lorsqu'il est laissé activé sur un endpoint exposé, il remet la même carte à un attaquant, supprimant le seul obstacle que le cadrage binaire place autrement devant la découverte.

## Énumérer la surface

Avec la réflexion activée, un seul appel d'outil liste chaque service et méthode, puis les types de messages de chacun. `grpcurl` le pilote directement :

```
grpcurl -plaintext target:50051 list
grpcurl -plaintext target:50051 list admin.AdminService
grpcurl -plaintext target:50051 describe admin.AdminService.DeleteTenant
```

`list` renvoie les services, `list <service>` les méthodes, et `describe` les formes des messages de requête et de réponse. À partir de rien d'autre qu'un port joignable, cela produit la surface RPC complète, y compris les services internes ou d'administration qui n'étaient jamais censés être appelés par des clients externes et qui n'ont aucune autre documentation publique.

## Pourquoi cela importe au-delà du confort

Le gRPC binaire sans le `.proto` est véritablement difficile à explorer : un appelant doit deviner les noms de méthodes et les champs de messages octet par octet. La réflexion supprime entièrement ce coût, de sorte qu'un service dont la seule protection était l'obscurité, une méthode admin sur le même serveur, un RPC de débogage, une interface de gestion interne, devient trivialement appelable une fois son nom et la forme de son message connus. La découverte n'est pas la compromission en soi, mais c'est l'étape qui rend praticable toute autre attaque gRPC (appeler une méthode que l'appelant ne devrait pas, élaborer un message valide pour elle). Après énumération, invoquez une méthode découverte avec une requête élaborée :

```
grpcurl -plaintext -d '{"tenant_id":"104"}' target:50051 admin.AdminService.DeleteTenant
```

## Confirmer l'exposition

Pointez `grpcurl ... list` sur l'endpoint. Une liste de services peuplée confirme que la réflexion est activée et joignable ; une erreur `Unimplemented` pour le service de réflexion signifie qu'elle est désactivée, et la surface doit alors être récupérée à partir des bundles clients, du trafic capturé ou de fichiers `.proto` ayant fuité. La réflexion est fréquemment activée dans les configurations de développement et laissée active en production, il vaut donc la peine de vérifier avant de supposer qu'un endpoint gRPC est opaque.

## Outils

- [grpcurl](https://github.com/fullstorydev/grpcurl)
- [grpcui](https://github.com/fullstorydev/grpcui)

## Références

- [gRPC: Server Reflection](https://grpc.io/docs/guides/reflection/)
- [OWASP API Security Top 10](https://owasp.org/API-Security/editions/2023/en/0x11-t10/)
