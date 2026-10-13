---
title: "Injection de template côté serveur Twig"
description: "Exploiter la SSTI Twig : confirmation avec {{7*7}}, accès à l'exécution de commande via les callbacks filter et map, et la chaîne historique _self registerUndefinedFilterCallback."
keywords:
  - SSTI Twig
  - Symfony
  - filter system
  - registerUndefinedFilterCallback
  - _self
---

# Twig

Twig est le moteur par défaut dans Symfony et est largement utilisé en autonome. Confirmez l'injection avec `{{7*7}}` renvoyant `49`. Twig n'a pas d'`import` direct ni de parcours d'attributs vers PHP comme les moteurs Python atteignent `os` ; à la place, la RCE vient des filtres et fonctions qui remettent une chaîne de caractères à un callable PHP.

La voie moderne fiable est le filtre `filter` ou `map`, qui applique une fonction PHP nommée à chaque élément. Le pointer vers `system` (ou `exec`, `passthru`) exécute une commande :

```twig
{{ ['id'] | filter('system') }}
{{ ['id'] | map('system') | join }}
```

`filter` itère le tableau et appelle `system('id')` sur l'élément. Lorsqu'une valeur unique est plus propre, le filtre `reduce` prend aussi une fonction fléchée qui peut appeler une fonction, et du code plus ancien peut exposer les callbacks `sort`/`map` de la même manière.

Sur Twig 1.x, la chaîne historique utilisait l'objet `_self` et `registerUndefinedFilterCallback` / `registerUndefinedFunctionCallback` pour enregistrer `system` ou `call_user_func` comme gestionnaire d'un filtre inconnu, puis l'invoquait :

```twig
{{ _self.env.registerUndefinedFilterCallback('system') }}{{ _self.env.getFilter('id') }}
{{ _self.env.registerUndefinedFilterCallback('exec') }}{{ _self.env.getFilter('cat /etc/passwd') }}
```

Dans Twig 2.x et ultérieur, `_self` ne renvoie que le nom du template, donc cette chaîne est fermée et la forme `filter('system')` est celle à utiliser.

Lorsque l'application exécute Twig via son extension de sandbox (`SandboxExtension`), les filtres et fonctions sont mis sur liste blanche et `system` n'est pas appelable ; `{{7*7}}` s'affiche toujours mais les callbacks sont bloqués. Testez un filtre connu comme restreint pour détecter le sandbox, et lorsqu'il est présent, l'injection est limitée à la divulgation d'information (lecture de variables telles que l'objet `app` de Symfony) plutôt qu'à la RCE. Le propre contexte de Symfony expose fréquemment `app.request`, l'environnement et les données de session même sous le sandbox, donc énumérez-les avant de conclure que la découverte est à faible impact.

## Outils

- tplmap, SSTImap

## Références

- Documentation Twig : filter, map, sandbox extension
- Avis de sécurité Symfony (contournements de sandbox)
- PortSwigger Web Security Academy: Server-side template injection
