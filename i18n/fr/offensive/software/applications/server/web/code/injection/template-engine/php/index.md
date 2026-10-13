---
order: 4
title: "Injection de template côté serveur en PHP"
description: "SSTI dans les moteurs de template PHP : la RCE par callback de filtre de Twig (Symfony) et les constructions d'exécution PHP de Smarty."
keywords:
  - SSTI PHP
  - Twig
  - Smarty
  - injection de template Symfony
---

# PHP

Les deux moteurs de template courants de PHP mènent tous deux à l'exécution de code depuis une injection de template, mais par des voies différentes. Twig (le moteur par défaut dans Symfony et largement utilisé en autonome) s'atteint via ses callbacks de filtre et de fonction ; Smarty expose PHP plus directement via des constructions dédiées qui se sont resserrées au fil des versions.

Détectez Twig avec `{{7*7}}` renvoyant `49`, et Smarty avec `{$smarty.version}` renvoyant la chaîne de version (Smarty utilise des accolades simples, donc `{7*7}` n'est pas une sonde fiable et peut provoquer une erreur).

## Moteurs

- **[Twig](twig.md)** : les callbacks `filter`/`map` vers `system`, et le `_self` registerUndefinedFilterCallback historique.
- **[Smarty](smarty.md)** : le tag `{php}` et les constructions à méthode statique, avec leur conditionnement par version.

## Références

- Documentation Twig et Smarty (sandbox, security policy)
- PortSwigger Web Security Academy: Server-side template injection
