---
title: "Injection Expr : manipulation de logique et méthodes exposées par l'hôte dans expr-lang"
description: "Le langage expr-lang est sandboxé et ne peut pas appeler du Go arbitraire, donc une expression injectée manipule la décision de policy ou de logique qu'elle pilote et ne peut atteindre des effets de bord qu'à travers les méthodes des objets que l'hôte a placés dans l'environnement."
keywords:
  - expr-lang
  - injection expr Go
  - expression sandboxée
  - méthodes env
  - contournement de policy
  - manipulation de logique
---

# Expr

`expr` (`github.com/expr-lang/expr`) évalue une expression contre un environnement fourni par l'hôte et renvoie une valeur, typiquement un booléen pour une policy ou un nombre pour un calcul. Il est sandboxé par conception : pas d'imports de packages, pas d'appels Go arbitraires, pas de primitives de système de fichiers ou de réseau dans le langage lui-même. Une chaîne `expr` injectée ne peut pas lancer un processus comme le peut un `eval` de scripting. Ce qu'elle peut faire, c'est décider, avec un contrôle total, le résultat sur lequel l'application agit, et atteindre toute méthode que l'hôte a exposée sur les objets de l'environnement.

## L'environnement est la surface

Un programme `expr` compile et s'exécute contre un `env`, généralement une struct ou une map. L'expression peut lire ces champs, indexer ces maps et appeler les méthodes de ces valeurs. Cet ensemble est exhaustif ; rien n'est atteignable que l'hôte n'ait placé dans le scope.

```go
env := map[string]any{
    "user":   user,       // struct with exported fields and methods
    "amount": amount,
}
program, _ := expr.Compile(userExpression, expr.Env(env))
out, _ := expr.Run(program, env)
```

Ici l'expression peut lire `user.Role`, `amount`, et appeler toute méthode exportée sur `user`. Énumérer ces noms et méthodes est la première étape sur n'importe quelle cible.

## Manipulation de logique et de policy

Là où le résultat conditionne une décision, contrôler l'expression contrôle la décision. C'est l'impact le plus courant et le plus fiable.

```
# Force an allow decision
true
user.Role == "admin" or true

# Subvert a numeric rule
amount * 0
amount < 999999999
```

Si l'expression évaluée est le contrôle d'autorisation, la règle de remise ou la condition d'alerte, celles-ci les inversent directement quelles que soient les données réelles.

## Atteindre des effets de bord via les méthodes

La seule voie vers un comportement au-delà du calcul d'une valeur est une méthode sur un objet de l'environnement qui effectue elle-même des I/O ou de l'exécution. `expr` invoquera les méthodes exportées sur les valeurs dans le scope, donc si l'hôte a exposé un type dont les méthodes touchent une base de données, le système de fichiers ou une commande, ces méthodes sont appelables depuis l'expression avec des arguments choisis par l'attaquant.

```
# Only reachable if the host exposed such a type/method in env:
user.LoadProfile("../../etc/passwd")
svc.Run("id")
```

Ni l'une ni l'autre n'est une fonctionnalité d'`expr` ; chacune n'existe que parce que l'hôte a placé cet objet dans `env`. Le cadrage honnête pour un rapport est que l'exécution de code n'est pas inhérente au langage et dépend entièrement de l'environnement exposé, donc le travail consiste à inventorier les méthodes dans le scope et à tester lesquelles ont des effets de bord.

## Confirmer l'évaluation et l'abus de ressources

Confirmez que la chaîne est évaluée avec une arithmétique qui n'est pas un écho.

```
6121 * 8
```

Un `48968` renvoyé prouve l'évaluation. `expr` expose aussi des builtins tels que `map`, `filter` et les opérateurs de range ; un grand range ou une compréhension imbriquée pilote un travail CPU et mémoire pendant l'évaluation, donnant un levier de déni de service quand aucun impact de logique ou de méthode n'est disponible.

```
len(filter(1..10000000, {# > 0}))
```

## Outils

- Tests manuels avec Burp Repeater ; payloads façonnés par moteur.
- **Burp Intruder** : énumérer les variables d'environnement atteignables et les méthodes exportées.

## Références

- [expr-lang documentation](https://expr-lang.org/)
- [expr-lang project](https://github.com/expr-lang/expr)
- [OWASP: Expression Language Injection](https://owasp.org/www-community/vulnerabilities/Expression_Language_Injection)
