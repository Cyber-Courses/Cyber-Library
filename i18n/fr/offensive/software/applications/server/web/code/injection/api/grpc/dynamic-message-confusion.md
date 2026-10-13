---
order: 3
title: "Confusion de messages dynamiques gRPC"
description: "Lorsqu'un serveur décode google.protobuf.Any ou résout un oneof selon un type choisi par l'attaquant, il peut être orienté pour désérialiser des octets dans un handler non prévu, un confused deputy entre types de messages."
keywords:
  - protobuf Any
  - type URL
  - confusion oneof
  - message dynamique
  - registre protobuf
---

# Confusion de messages dynamiques

Protobuf est normalement rigidement typé, mais quelques fonctionnalités réintroduisent le dispatch dynamique : `google.protobuf.Any` porte une type URL plus des octets opaques à désérialiser dans le type que l'URL nomme, `oneof` permet à un champ de contenir l'un de plusieurs types, et des registres résolvent les noms de types à l'exécution. Lorsque le serveur choisit comment décoder un message à partir d'une valeur que l'appelant contrôle, un attaquant choisit le type dans lequel les octets sont décodés, orientant la requête vers un handler qui n'était pas censé la traiter.

## Choisir le type avec Any

Un champ `Any` diffère la décision de la nature du payload jusqu'au moment de la désérialisation, selon sa `type_url`. Si le serveur désérialise sans restreindre les types autorisés, l'appelant en choisit un :

```json
{
  "payload": {
    "@type": "type.googleapis.com/internal.AdminCommand",
    "action": "promote",
    "target": "104"
  }
}
```

Un handler qui accepte un `Any` générique à des fins bénignes, un événement, une pièce jointe, un blob de métadonnées, et le désérialise ensuite dans un type choisi par l'expéditeur, peut être amené à matérialiser une commande interne ou un type de message privilégié, sur lequel le code en aval agit ensuite comme s'il était de confiance.

## oneof et résolution par registre

Un `oneof` ne contient que les membres que son `.proto` déclare, donc sélectionner un membre différent est un comportement conforme au contrat, pas du typage dynamique en soi. Cela devient une faille lorsque les branches ne sont pas soumises aux mêmes vérifications : si une branche porte une étape d'autorisation ou de validation qu'une autre saute, choisir la branche la plus faible atteint l'action sans le contrôle que le chemin commun imposait. Le cas véritablement dynamique est la résolution par registre, où le serveur recherche un message ou un handler par un nom pris dans la requête ; fournir un nom inattendu atteint une implémentation différente que le contrat n'a jamais fixée. Le cas `Any` ci-dessus et les recherches par registre sélectionnent le type ou le handler à partir d'une entrée non fiable ; un `oneof` ne compte que là où une branche est sous-protégée par rapport à ses semblables.

## Pourquoi le typage fort ne l'empêche pas

La promesse de protobuf est qu'un champ a un seul type, de sorte que des octets non fiables ne peuvent pas être réinterprétés. `Any`, `oneof` et les registres brisent cette promesse délibérément, pour prendre en charge l'extensibilité, et ce faisant recréent le problème classique du « décoder les octets de l'attaquant comme un type choisi par l'attaquant ». Le serveur doit fixer l'ensemble des types qu'il désérialisera et rejeter les valeurs `type_url` inconnues ; là où il ne le fait pas, le choix de type de l'appelant est la vulnérabilité. Le test consiste à envoyer un message externe valide dont le `Any` ou le `oneof` sélectionne un type interne ou inattendu, et à observer si le serveur le désérialise et agit dessus plutôt que de rejeter le type.

## Outils

- **grpcurl** : envoi d'un message élaboré dont le Any ou le oneof sélectionne un type inattendu.
- **blackboxprotobuf (extension Burp)** : édition des champs protobuf dans le trafic gRPC intercepté.
- **grpcui** : exécution interactive des méthodes contre un serveur avec réflexion.

## Références

- [Protocol Buffers: Any](https://protobuf.dev/programming-guides/proto3/#any)
- [OWASP API Security Top 10](https://owasp.org/API-Security/editions/2023/en/0x11-t10/)
