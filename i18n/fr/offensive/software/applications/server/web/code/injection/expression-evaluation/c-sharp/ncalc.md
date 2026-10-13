---
title: "Injection NCalc : abuser d'un évaluateur de math et de logique .NET"
description: "NCalc évalue l'arithmétique, la logique et les fonctions enregistrées mais n'offre par lui-même ni réflexion ni exécution de code arbitraire, donc la surface injectable est la grammaire, les hooks EvaluateParameter et EvaluateFunction, et toute fonction personnalisée dangereuse."
keywords:
  - NCalc
  - évaluateur math .NET
  - EvaluateFunction
  - EvaluateParameter
  - RCE par fonction personnalisée
  - manipulation de logique
---

# NCalc

NCalc est un évaluateur d'expressions mathématiques et logiques pour .NET. Il parse l'arithmétique, les comparaisons, la logique booléenne et un ensemble de fonctions intégrées et enregistrées par l'hôte, et renvoie la valeur calculée. Il est important d'être précis sur ce que NCalc standard ne fait pas : par lui-même il n'offre ni réflexion, ni références de types, ni exécution .NET arbitraire. Une expression NCalc injectée ne peut pas atteindre `Process.Start` comme le peut un évaluateur compilé. La vraie surface d'attaque est la grammaire qu'il expose bel et bien, les hooks de l'hôte autour, et toutes les fonctions que l'application a choisi d'enregistrer.

## La grammaire comme surface

NCalc évalue la grammaire arithmétique et logique complète, y compris les comparaisons, les conditionnelles et l'ensemble de fonctions intégrées (`if`, `in`, `Abs`, `Pow`, les helpers de chaîne et similaires). Cette grammaire suffit à subvertir toute décision que l'application dérive du résultat.

```
# Force a rule or authorization branch
1 == 1
true or false

# Skew a computed value
basePrice * 0
if(true, 0, basePrice)
```

Là où le résultat évalué alimente un prix, une remise, un seuil ou une décision autoriser/refuser, contrôler l'expression contrôle ce résultat directement. C'est de la manipulation de logique plutôt que de l'exécution de code, et c'est fréquemment tout l'impact sur une cible donnée.

## Les hooks de l'hôte

NCalc déclenche deux événements que l'hôte câble : `EvaluateParameter`, qui résout les paramètres nommés, et `EvaluateFunction`, qui résout les appels de fonctions que le moteur ne reconnaît pas. Ces handlers sont du code applicatif, et une expression injectée choisit quels paramètres et fonctions invoquer.

```csharp
var e = new Expression(userInput);
e.EvaluateFunction += (name, args) => {
    if (name == "lookup") args.Result = Db.Lookup((string)args.Parameters[0].Evaluate());
};
```

Une expression peut appeler `lookup(...)` avec des arguments choisis par l'attaquant, passer des valeurs que le développeur n'avait jamais prévues, et chaîner des fonctions enregistrées. Le comportement atteignable est exactement l'ensemble des paramètres et fonctions que l'hôte résout, alors énumérez-les : chaque nom de fonction personnalisée qui s'évalue proprement est appelable avec des arguments contrôlés.

## La seule voie d'exécution de code

NCalc n'atteint l'exécution de code que lorsque l'application a enregistré une fonction personnalisée qui fait elle-même quelque chose de dangereux. Une fonction qui invoque un shell, lit des fichiers, interroge une base de données ou réfléchit sur des types devient le payload, et NCalc n'est que le moyen de l'appeler avec l'entrée de l'attaquant.

```
# Only if the host registered such a function:
exec('whoami')
readfile('C:\\inetpub\\wwwroot\\web.config')
```

Ni l'une ni l'autre n'existe dans le NCalc de base. Leur présence est une propriété de la cible, donc l'évaluation consiste à découvrir l'ensemble des fonctions enregistrées et à tester chacune pour des effets de bord, plutôt qu'à supposer une RCE NCalc générique qui n'existe pas.

## Confirmer l'évaluation et l'abus de ressources

Prouvez que l'entrée est évaluée avec une arithmétique qui n'est pas un écho fortuit.

```
9134 * 7
```

Un `63938` renvoyé confirme que la chaîne est parsée comme une expression NCalc. Au-delà de l'abus de logique et de fonctions, la grammaire prend aussi en charge l'abus de ressources : une expression arithmétique profondément imbriquée ou volumineuse force un travail CPU et mémoire pendant le parsing et l'évaluation, donnant un levier de déni de service là où aucun impact plus fort n'est atteignable.

## Outils

- Tests manuels avec Burp Repeater ; payloads façonnés par moteur.
- **Burp Intruder** : énumérer les paramètres et fonctions personnalisées enregistrés par leur nom.

## Références

- [NCalc documentation](https://ncalc.github.io/ncalc/)
- [NCalc project](https://github.com/ncalc/ncalc)
- [OWASP: Expression Language Injection](https://owasp.org/www-community/vulnerabilities/Expression_Language_Injection)
