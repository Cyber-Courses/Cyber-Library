---
order: 7
title: "Injection de template côté serveur Ruby"
description: "SSTI dans les moteurs de template Ruby : ERB (et les variantes compatibles Erubi/Erubis) qui évaluent directement du Ruby embarqué pour une exécution de commande immédiate."
keywords:
  - SSTI Ruby
  - ERB
  - Erubi
  - injection de template Rails
---

# Ruby

La SSTI en Ruby tourne autour d'ERB, le moteur embarqué standard de Ruby utilisé par Rails et de nombreuses applications autonomes (avec Erubi et Erubis comme variantes compatibles). ERB évalue du Ruby à l'intérieur de ses balises sans sandbox, si bien qu'une injection de template est une exécution directe de code : il n'y a pas de graphe d'objets à parcourir ni de built-in à contourner.

La détection utilise la propre syntaxe de balises d'ERB plutôt que `{{7*7}}`, puisqu'ERB exécute `<%= ... %>`. L'escalade est immédiate via les backticks de Ruby, `system`, `IO.popen` ou `%x{}`.

## Moteurs

- **[ERB](erb.md)** : `<%= `command` %>` et les variantes `system`/`IO.popen`, ainsi que le sink Rails `render inline:`.

## Références

- Documentation Ruby ERB et Erubi
- PortSwigger Web Security Academy: Server-side template injection
