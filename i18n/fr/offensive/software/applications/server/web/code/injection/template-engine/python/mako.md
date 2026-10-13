---
title: "Injection de template côté serveur Mako"
description: "Exploitation d'une SSTI Mako : Mako évalue directement du Python à l'intérieur de ses blocs d'expression et de code, si bien qu'une injection de template devient immédiatement une exécution de code à distance."
keywords:
  - SSTI Mako
  - injection de template Python
  - exécution directe de code
  - bloc de niveau module
---

# Mako

Mako (utilisé par Pylons, Pyramid et quelques applications autonomes) diffère de Jinja2 sur un point crucial : il n'a pas de sandbox et évalue du Python arbitraire dans ses constructions d'expression et de code. Une injection de template est donc une exécution directe de code, sans aucun des parcours de graphe d'objets dont Jinja2 a besoin.

Confirmez avec `${7*7}` qui renvoie `49`. Les blocs d'expression évaluent n'importe quelle expression Python, si bien qu'importer et lancer une commande tient sur une seule ligne :

```mako
${__import__('os').popen('id').read()}
```

Les blocs de contrôle de Mako exécutent des instructions, ce qui permet un import puis un appel plus propres :

```mako
<% import os %>${os.popen('id').read()}
```

Les blocs de niveau module (`<%! ... %>`) s'exécutent une seule fois au moment de l'import et sont tout aussi utilisables. Mako expose également des helpers dont les attributs atteignent le système d'exploitation, par exemple `${self.module.cache.util.os.system('id')}`, mais la forme directe `__import__('os')` reste la plus simple et la plus fiable.

Comme l'évaluation n'est pas restreinte, il n'y a pas de sandbox à contourner ; les seuls obstacles sont les filtres d'entrée côté application (qui bloquent `${`, `import` ou les backticks), que l'on contourne grâce aux syntaxes de blocs alternatives de Mako et aux nombreuses façons qu'a Python d'obtenir un module (`__import__`, `importlib`, `os` via `sys.modules`). La conclusion est que confirmer Mako revient à confirmer une RCE : détection et exploitation se confondent en une seule étape.

## Outils

- tplmap, SSTImap

## Références

- Documentation de Mako : syntaxe d'expression et de contrôle
- PortSwigger Web Security Academy: Server-side template injection
