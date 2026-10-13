---
order: 4
title: "Abus de JSON-RPC : dispatch de méthode et de paramètres"
description: "JSON-RPC mappe un nom de méthode et des params sur une procédure serveur. La surface d'attaque est la façon dont la chaîne de méthode se résout en fonction, dont la forme des params est gérée, et dont les tableaux groupés sont autorisés."
keywords:
  - injection JSON-RPC
  - dispatch de méthode
  - structure des params
  - requête groupée
  - allowlist de procédures
---

# JSON-RPC

JSON-RPC est minimal : une requête nomme une `method`, porte des `params`, et le serveur mappe le nom sur une procédure et l'appelle avec ces params. Cette légèreté renvoie toutes les décisions de sécurité dans le code applicatif, comment la chaîne de méthode devient une fonction, comment les params sont convertis en arguments, et comment un groupe d'appels est autorisé, et chacune est un endroit où une implémentation laxiste permet à un appelant d'atteindre plus que prévu.

## Les trois surfaces

- **[Contournement du nom de procédure](procedure-name-bypass.md)** : la chaîne `method` mappée sur une procédure via une réflexion non sûre, des astuces de casse, de la contrebande de délimiteurs ou du retrait de préfixe, atteignant des méthodes internes ou admin que l'allowlist croyait exclure.
- **[Abus de la structure des paramètres](parameter-structure-abuse.md)** : `params` accepté de façon incohérente comme tableau ou objet, de sorte que des clés manquantes se convertissent en valeurs par défaut, que d'autres chemins de code se déclenchent, ou que des parseurs laxistes laissent fuir du routage dans les erreurs.
- **[Abus des requêtes groupées](batch-request-abuse.md)** : un tableau groupé où une entrée échoue l'autorisation en mode ouvert tandis qu'une autre exécute une méthode sensible, exploitant le traitement par élément et la politique d'erreur.

## Pourquoi le minimalisme déplace le risque vers le code

Il n'y a pas de framework de routes, de verbes et de négociation de contenu sur lequel s'appuyer ; un serveur JSON-RPC est une table de dispatch et un lieur de paramètres écrits par l'application. Si cette table est construite avec une recherche d'attribut dynamique au lieu d'une carte explicite, le nom de méthode atteint plus de fonctions que prévu. Si le lieur devine la forme des params au lieu de la valider, l'appelant choisit le chemin de code. Le test consiste à sonder comment un nom de méthode se résout et comment les params sont analysés, avant de supposer que l'un ou l'autre est contraint.

## Outils

- **Burp Suite** : interception et rejeu des payloads de méthode et de params JSON-RPC.
- **curl** : scripting de sondes de nom de méthode et de forme de params contre l'endpoint.

## Références

- [JSON-RPC 2.0 Specification](https://www.jsonrpc.org/specification)
- [OWASP API Security Top 10](https://owasp.org/API-Security/editions/2023/en/0x11-t10/)
