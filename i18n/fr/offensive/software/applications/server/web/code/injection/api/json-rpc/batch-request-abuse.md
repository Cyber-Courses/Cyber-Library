---
order: 2
title: "Abus des requêtes groupées JSON-RPC"
description: "Un tableau groupé où une entrée échoue l'autorisation en mode ouvert tandis qu'une autre exécute une méthode sensible exploite le traitement par élément et la politique d'erreur pour faire passer des appels privilégiés au-delà des contrôles par requête."
keywords:
  - groupage JSON-RPC
  - autorisation de groupe
  - multi-appel
  - autorisation par élément
  - échec atomique
---

# Abus des requêtes groupées

JSON-RPC permet à un client d'envoyer un tableau de requêtes en un seul appel, et le serveur traite chacune et renvoie un tableau de réponses. Lorsque des contrôles qui devraient s'appliquer à chaque appel, autorisation, limitation de débit, journalisation, sont implémentés une fois pour la requête HTTP au lieu d'une fois par entrée du tableau, le groupe devient un emballage qui transporte des appels privilégiés ou répétés sous une seule enveloppe autorisée.

## Contrôles par requête appliqués une seule fois

Une limite de débit ou un compteur de tentatives qui s'incrémente par requête HTTP ne voit qu'une seule requête quel que soit le nombre d'entrées que contient le groupe. Entasser de nombreux appels sensibles dans un seul groupe les exécute tous sous un seul tic du compteur :

```json
[
  {"jsonrpc":"2.0","method":"redeemCoupon","params":{"code":"A1"},"id":1},
  {"jsonrpc":"2.0","method":"redeemCoupon","params":{"code":"A2"},"id":2},
  {"jsonrpc":"2.0","method":"redeemCoupon","params":{"code":"A3"},"id":3}
]
```

C'est la forme JSON-RPC de l'amplification que les alias fournissent en GraphQL : une opération limitée s'exécute bien plus de fois que la limite par requête ne le prévoyait.

## Autorisation qui échoue en mode ouvert en plein groupe

Lorsque l'autorisation est vérifiée pour le groupe dans son ensemble, ou lorsqu'une entrée en échec n'arrête pas les autres, un attaquant associe un appel autorisé à un appel restreint. Si le serveur autorise sur la première entrée, ou continue au-delà d'une entrée non autorisée et exécute tout de même les suivantes, l'appel sensible s'exécute :

```json
[
  {"jsonrpc":"2.0","method":"getPublicStatus","params":[],"id":1},
  {"jsonrpc":"2.0","method":"admin.deleteUser","params":{"id":104},"id":2}
]
```

Un premier appel bénin établit un contexte permissif ou passe une vérification grossière, et le second s'exécute car l'autorisation par entrée n'a jamais été appliquée.

## Fuites par les erreurs et l'ordonnancement

Les groupes laissent aussi fuir du comportement via l'ordonnancement et les erreurs. Observer quelles entrées réussissent, lesquelles échouent et comment les erreurs sont signalées révèle si l'autorisation, la validation et les effets de bord sont évalués par entrée ou une fois pour le tableau, ce qui guide l'endroit où placer l'appel restreint.

## Confirmer la faille

Envoyez un groupe dont les entrées seraient limitées ou bloquées individuellement, et comparez le résultat à l'envoi une par une. Si une limite par requête s'enregistre une seule fois pour tout le groupe, ou si une méthode restreinte s'exécute lorsqu'elle est associée à une méthode autorisée, les contrôles sont appliqués à la mauvaise granularité. Le correctif, autoriser et comptabiliser chaque entrée indépendamment et définir une politique d'échec atomique, est la propriété que ce test cherche.

## Outils

- **Burp Repeater** : assemblage de tableaux groupés et comparaison des résultats par entrée.
- **curl** : envoi de requêtes JSON-RPC groupées pour comparer aux appels uniques.

## Références

- [JSON-RPC 2.0 Specification: Batch](https://www.jsonrpc.org/specification#batch)
- [OWASP API Security Top 10: Unrestricted Resource Consumption](https://owasp.org/API-Security/editions/2023/en/0xa4-unrestricted-resource-consumption/)
