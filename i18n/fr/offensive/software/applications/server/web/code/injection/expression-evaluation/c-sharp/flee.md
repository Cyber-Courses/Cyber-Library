---
title: "Injection Flee : expressions compilées en IL atteignant les types .NET importés"
description: "Flee compile les expressions en IL et appelle de vraies méthodes .NET sur les types de ses Imports et de son contexte owner, donc une expression injectée atteint l'exécution de code dans la limite de la surface de types importés."
keywords:
  - Flee
  - Fast Lightweight Expression Evaluator
  - injection d'expression .NET
  - ExpressionContext Imports
  - compilation IL
  - appel de méthode statique
---

# Flee

Flee, le Fast Lightweight Expression Evaluator, n'est pas un interpréteur de chaînes. Il compile chaque expression en IL et l'exécute comme du vrai code .NET. Cette compilation est ce qui distingue Flee d'un évaluateur purement mathématique : une expression peut référencer les types .NET que l'hôte a rendus disponibles et appeler leurs méthodes et propriétés statiques. Lorsque le texte de l'expression est contrôlé par l'attaquant, l'impact atteignable est tout ce que ces types exposent, jusqu'à et y compris l'exécution de code.

## Ce que l'expression peut atteindre

Une expression Flee est évaluée contre un `ExpressionContext`. Deux parties de ce contexte décident de la surface d'attaque :

- **Imports** (`context.Imports`) : les types et namespaces que l'hôte a enregistrés pour que l'expression puisse les nommer. Un type importé est appelable par son nom dans l'expression.
- **L'objet owner** (`context.Imports.AddInstance` / l'owner passé au contexte) : ses membres publics sont atteignables sans qualification.

Flee se lie à des membres réels et émet de l'IL qui les invoque. Donc la question sur une cible donnée n'est pas de savoir si Flee peut appeler des méthodes (il le peut) mais quels types sont dans le scope.

```csharp
var context = new ExpressionContext();
context.Imports.AddType(typeof(Math));
var e = context.CompileGeneric<double>(userExpression);
double result = e.Evaluate();
```

Avec seulement `Math` importé, `userExpression` atteint `Math.Sqrt`, `Math.Pow` et la grammaire arithmétique. La surface reste juste ce type.

## L'escalade dépend de la surface importée

Dès que l'hôte importe un type qui expose une méthode dangereuse, une expression injectée peut l'appeler. L'exemple canonique est un contexte qui a `System.Diagnostics.Process` dans le scope, ce qui transforme l'évaluateur en lanceur de processus.

```
# With System.Diagnostics.Process imported:
Process.Start("cmd.exe", "/c whoami")

# With System.IO.File imported:
File.ReadAllText("C:\\inetpub\\wwwroot\\web.config")
```

Même sans import manifestement dangereux, un type amené par commodité peut être un pivot : tout ce qui expose une propriété statique renvoyant un objet plus riche, une factory, ou une méthode qui touche le système de fichiers ou la table des processus étend la portée. Énumérez d'abord les imports, car ils constituent la frontière exacte de ce que l'injection peut faire.

## Confirmer la compilation et sonder le scope

Commencez par prouver que l'entrée est compilée comme une expression Flee, avec une arithmétique qui ne peut pas être un écho fortuit.

```
8123477 * 3
```

Un `24370431` renvoyé confirme l'évaluation. Ensuite, sondez quels types sont dans le scope en les nommant et en lisant un membre statique inoffensif ; une compilation réussie signifie que le type est importé et que ses membres sont appelables.

```
# Does the context import these? A clean evaluate answers yes.
DateTime.Now
Environment.MachineName
```

Une fois qu'un type atteignable avec une méthode utile est confirmé, appelez-la. Si rien de dangereux n'est importé, Flee reste une surface arithmétique et logique : inversez une règle booléenne, faussez un prix calculé, ou pilotez un calcul coûteux pour de l'abus de ressources. Le passage à l'exécution de code est entièrement conditionné par la liste des imports, donc rapportez l'impact par rapport aux types réellement en contexte plutôt que de supposer le pire.

## Outils

- Tests manuels avec Burp Repeater ; payloads façonnés par moteur.
- **Burp Intruder** : sonder quels types .NET sont importés dans l'ExpressionContext.

## Références

- [Flee project (Fast Lightweight Expression Evaluator)](https://github.com/mparlak/Flee)
- [OWASP: Expression Language Injection](https://owasp.org/www-community/vulnerabilities/Expression_Language_Injection)
