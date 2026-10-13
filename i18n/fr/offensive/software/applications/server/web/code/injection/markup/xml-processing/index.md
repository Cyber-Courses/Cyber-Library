---
title: "Injection dans le traitement XML"
description: "Injection à travers tout le pipeline XML : parsing, interrogation avec XPath/XQuery et transformation avec XSLT, ainsi que l'abus d'entités externes que les parseurs permissifs autorisent."
keywords:
  - injection XML
  - injection XPath
  - injection XQuery
  - injection XSLT
  - XXE
---

# Traitement XML

Un pipeline XML fait passer un document par trois étapes, et chacune est une surface d'injection distincte. Le document est **parsé** en un arbre, il est **interrogé** pour en extraire des valeurs, et il peut être **transformé** en un autre format. L'entrée utilisateur concaténée dans l'une quelconque de ces étapes, plutôt que liée en tant que donnée, est interprétée comme faisant partie du jeu d'instructions que cette étape exécute.

À l'étape de **parsing**, un parseur configuré pour résoudre les entités externes transforme un document forgé en XXE, lisant des fichiers locaux, atteignant des services internes et épuisant les ressources. À l'étape d'**interrogation**, une expression **XPath** ou **XQuery** construite par concaténation de chaînes se comporte comme une injection SQL : l'entrée s'échappe de son littéral de chaîne et réécrit les prédicats et les chemins de localisation pour sélectionner des nœuds non prévus ou contourner l'authentification. À l'étape de **transformation**, les processeurs **XSLT** exposent leurs propres fonctions, et un fragment de feuille de style injecté peut lire des fichiers, appeler des fonctions d'extension ou atteindre le réseau.

La cause racine commune est que XML brouille la frontière entre donnée et instruction plus facilement que la plupart des formats, parce que les entités, les chemins de localisation et les directives de feuille de style circulent tous à l'intérieur du même document que l'application traite comme une entrée.

Cette section couvre en profondeur l'injection de requêtes XPath, aux côtés des attaques plus larges de parsing et de transformation XML.

## Sous-thèmes

- **[XPath](xpath/index.md)** : une expression XPath construite par concaténation de chaînes sur un document XML permet à un attaquant de réécrire les prédicats et les chemins de localisation pour contourner l'authentification, sélectionner des nœuds non prévus et extraire...
- **[XQuery](xquery/index.md)** : une expression XQuery construite par concaténation de chaînes contre des bases de données XML permet à un attaquant de réécrire les expressions de chemin et les prédicats, d'atteindre le système de fichiers et le réseau via des fonctions intégrées...
- **[XSLT](xslt/index.md)** : des feuilles de style ou des paramètres de transformation influencés par l'attaquant lui permettent de lire des fichiers, d'atteindre des services internes, de récupérer des feuilles de style distantes et d'exécuter du code dans le moteur de transformation...
- **[XXE](xxe/index.md)** : les parseurs XML qui résolvent les DTD et les entités externes permettent à un attaquant de lire des fichiers locaux, d'atteindre des services internes et d'amplifier l'entrée en déni de service.

## Outils

- **Burp Suite** : analyse et forge de charges utiles à travers les étapes de parsing, d'interrogation et de transformation XML.
- **XXEinjector** : automatisation de la récupération de fichiers XXE et de l'exfiltration hors bande.
- **xcat** : automatisation de l'extraction de données XPath.
- **Burp Collaborator** : confirmation des attaques XML aveugles et hors bande.

## Références

- PortSwigger Web Security Academy : XML external entity (XXE) injection
- OWASP WSTG : Testing for XML Injection
