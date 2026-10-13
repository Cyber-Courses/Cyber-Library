---
order: 8
title: "Injection de template côté serveur (SSTI)"
description: "Exploiter l'injection de template côté serveur : détecter le moteur, escalader d'une expression de template vers le runtime du langage hôte, et les limites de sandbox qui décident si la RCE est accessible, organisé par langage."
keywords:
  - injection de template côté serveur
  - SSTI
  - moteur de template
  - évasion de sandbox
  - RCE via template
---

# Moteur de template

L'injection de template côté serveur (SSTI) se produit lorsqu'une entrée utilisateur est embarquée dans un template que le serveur rend ensuite, de sorte que l'entrée est évaluée comme du code de template plutôt que comme des données. Parce que les templates peuvent atteindre le langage hôte, la SSTI escalade souvent d'une sortie reflétée à une exécution de code à distance, le plafond exact étant fixé par le moteur et son sandbox.

La première étape est de détecter le moteur, car chacun a sa propre syntaxe d'expression. Une sonde mathématique qui s'affiche différemment selon le moteur permet de le cerner : `{{7*7}}` (renvoie `49`) pointe vers Jinja2, Twig ou Nunjucks ; `${7*7}` vers FreeMarker ou un contexte JSP/EL ; `#{7*7}` vers Pug ou un contexte JSF ; `<%= 7*7 %>` vers ERB ; `#set($x=7*7)$x` vers Velocity ; `@(7*7)` vers Razor ; `[% 7*7 %]` vers Template Toolkit. Un polyglotte tel que `${{<%[%'"}}%\` déclenche une erreur révélatrice dans plusieurs moteurs. Lorsque `7*7` s'affiche en `49`, l'entrée est évaluée, pas simplement reflétée.

Le schéma d'escalade est partagé : depuis une expression de template, parcourez le graphe d'objets ou les built-ins que le moteur expose pour atteindre le runtime du langage (un class loader, un dictionnaire de globals, un utilitaire qui exécute des commandes), puis appelez vers l'OS. Que cela réussisse dépend du sandbox : certains moteurs exécutent les templates non fiables en mode restreint par défaut (et la tâche est de s'en échapper), tandis que d'autres, et la plupart des moteurs lorsque le framework n'active pas le sandboxing, évaluent librement.

Ce domaine est organisé par langage, puis par moteur, puisque les gadgets sont spécifiques au moteur.

## Langages

- **[Python](python/index.md)** : Jinja2, Mako.
- **[PHP](php/index.md)** : Twig, Smarty.
- **[Java](java/index.md)** : FreeMarker, Velocity.
- **[JavaScript](javascript/index.md)** : Handlebars, Pug.
- **[Ruby](ruby/index.md)** : ERB.
- **[Go](go/index.md)** : text/template et html/template.
- **[C#](c-sharp/index.md)** : Razor.
- **[Rust](rust/index.md)** : Tera.
- **[Perl](perl/index.md)** : Template Toolkit.

## Références

- PortSwigger Web Security Academy: Server-side template injection
- OWASP Testing Guide: Testing for Server-Side Template Injection
