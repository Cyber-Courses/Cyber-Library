---
title: "Évasion de sandbox Python : récupérer les builtins et l'accès OS depuis un exec restreint"
description: "S'échapper d'un eval/exec Python restreint : parcourir le graphe des sous-classes d'objets pour atteindre os et subprocess, reconstruire __import__ et __builtins__, et déjouer les denylists de noms."
keywords:
  - évasion de sandbox python
  - __subclasses__
  - __builtins__
  - exec restreint
  - audit hooks
---

# Évasion de sandbox Python

Les tentatives d'exécuter du Python non fiable en toute sécurité, en retirant `__builtins__`, en interdisant par denylist des noms comme `os`/`eval`/`import`, ou en filtrant le source, se heurtent au fait que le modèle objet de Python garde tout le runtime accessible depuis presque n'importe quel objet. À partir d'une primitive `eval`/`exec` confinée (un template en sandbox, un endpoint « calculatrice », une SSTI qui atterrit dans un `SandboxedEnvironment`), vous remontez jusqu'à l'OS.

## Récupérer l'accès OS depuis le graphe d'objets

Même avec `__builtins__` retiré, tout objet expose sa hiérarchie de types, et quelque part dans l'arbre des sous-classes se trouvent des classes qui importent ou lancent des processus :

```python
# From any object, reach every loaded class:
().__class__.__base__.__subclasses__()
```

Indexez la liste résultante vers une classe utile, par exemple l'une de `subprocess.Popen`, `os._wrap_close`, ou une classe de warnings/loader dont les globals incluent `__builtins__`, puis appelez à travers elle :

```python
# Via a subclass whose __globals__ expose builtins/import:
[c for c in ().__class__.__base__.__subclasses__()
   if c.__name__ == 'catch_warnings'][0]()._module.__builtins__['__import__']('os').system('id')
```

Un motif général fiable consiste à trouver une fonction/classe avec `__globals__` et à en tirer `__builtins__['__import__']`, puis à importer `os`/`subprocess`.

## Déjouer les denylists de noms

Quand des chaînes précises sont filtrées, construisez-les indirectement :

- `getattr(obj, 'o'+'s')`, ou construire des noms à partir de `chr()`/concaténation.
- Atteindre des attributs aux noms encodés via `getattr`, ou via `__getattribute__`.
- Utiliser `__import__` obtenu depuis les globals au lieu de l'instruction `import`.

## SandboxedEnvironment de Jinja2

Le cas propre au web est le sandbox de Jinja2, où l'accès à un attribut commençant par `_` est vérifié par `is_safe_attribute` (donc encoder les underscores n'aide pas). L'évasion est une méthode que le sandbox autorise encore et qui fait fuiter le graphe d'objets, ce qui dépend de la version ; voir la page [SSTI Jinja2](../../code/injection/template-engine/python/jinja2.md) pour les spécificités du contexte de template. Le parcours des sous-classes ci-dessus est la primitive de niveau moteur qui le sous-tend.

## Notes d'exploitation

- `eval`/`exec` avec un `globals` restreint a toujours accès aux types des littéraux, donc les amorces de style `().__class__` fonctionnent même quand les noms sont retirés.
- Les **audit hooks** du CPython moderne (`sys.addaudithook`) et les vrais sandboxes (seccomp, interpréteur séparé, RestrictedPython) peuvent bloquer `os.system`/`subprocess` ; vérifiez leur présence et pivotez vers la lecture/l'écriture de fichiers ou le réseau via le module que les hooks permettent encore.
- Pour la sortie, préférez `subprocess.check_output(...).decode()` renvoyé via le sink, ou un callback OOB quand rien n'est affiché.

## Outils

- Payloads de parcours de sous-classes construits à la main ; SSTImap pour l'automatisation du contexte de template.

## Références

- Python data model: type/subclass introspection
- PortSwigger Web Security Academy: SSTI (sandbox escape)
