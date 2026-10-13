---
title: "Injection de data binding ASP.NET : contrôle d'expression Eval et DataBinder.Eval"
description: "Lorsque l'entrée de l'attaquant atteint la chaîne d'expression d'une balise de data binding telle que Eval ou DataBinder.Eval, elle s'évalue comme une expression .NET dans le contexte de la page et atteint l'exécution de code."
keywords:
  - data binding ASP.NET
  - DataBinder.Eval
  - injection d'expression Eval
  - expression de data binding
  - Web Forms
  - exécution de code .NET
---

# Data binding ASP.NET

Les expressions de data binding d'ASP.NET Web Forms, écrites `<%# ... %>`, sont évaluées lorsque le `DataBind()` d'un contrôle s'exécute. La forme classique est `<%# Eval("PropertyName") %>`, qui résout un chemin de propriété contre l'élément de données courant, et `DataBinder.Eval(Container.DataItem, "PropertyName")`, qui fait la même chose par programmation. Ce ne sont pas des substitutions de chaînes. Le contenu entre `<%#` et `%>` est une expression .NET compilée dans la classe de la page, donc tout ce qu'il contient s'exécute avec la pleine capacité du code de la page.

## La condition qui compte

Le cas dangereux est étroit et spécifique : l'attaquant doit contrôler le **texte** de l'expression, le contenu entre `<%#` et `%>`, et pas simplement une valeur liée à une propriété, ni simplement la chaîne passée en argument à `Eval`. Contrôler uniquement l'argument de `Eval` est faible : `Eval(attackerString)` traite toujours cette chaîne comme un chemin de navigation de propriété/indexeur contre l'élément de données, donc le plus qu'il produise est la lecture d'autres propriétés du graphe d'objets lié, et non l'exécution de code. L'exécution de code n'apparaît que lorsque l'application assemble elle-même le texte de l'expression à partir de l'entrée puis le compile.

```
# Benign: the value is data, the expression is fixed
<%# Eval("DisplayName") %>

# Weak: only the Eval ARGUMENT is attacker-controlled -> property-path
# navigation of the data item, not compiled code
<%# Eval(userSuppliedPath) %>

# Dangerous: the attacker controls the EXPRESSION TEXT that gets compiled
<%# {attacker-controlled .NET expression} %>
```

Lorsque le magasin inscriptible par l'attaquant alimente le texte de l'expression (un champ de template, une définition de rapport sauvegardée, du markup persistant, un fragment `.aspx` composé au runtime), ce texte est parsé et compilé comme une expression .NET contre la page, ce qui est la condition d'exécution de code.

## Atteindre le sink

Trois patterns placent du texte contrôlé par l'attaquant dans une expression évaluée :

- Une fonctionnalité de reporting ou de dashboard qui laisse les utilisateurs définir une colonne comme une expression, stockée puis émise plus tard dans un binding `<%# ... %>` ou passée à `DataBinder.Eval`.
- Un système de template de page ou de thème qui persiste du markup contenant des expressions de binding, où le markup persistant est inscriptible par l'attaquant.
- La génération dynamique de pages qui concatène l'entrée dans un fragment `.aspx` compilé au runtime.

Parce que l'expression se compile comme du code de page, elle n'est pas confinée à la résolution de propriétés. Une expression contrôlée peut instancier des types et appeler des méthodes atteignables depuis les imports de namespaces de la page.

```
<%# System.Diagnostics.Process.Start("cmd.exe","/c whoami") %>
<%# new System.IO.StreamReader("c:\\inetpub\\wwwroot\\web.config").ReadToEnd() %>
```

La première atteint le démarrage d'un processus ; la seconde lit un fichier depuis le contexte de l'application. Les deux s'exécutent sous l'identité de l'application pool, qui est le plafond d'impact ici.

## Confirmer l'évaluation

Prouvez que l'expression est compilée plutôt que renvoyée telle quelle avec une sonde arithmétique ou de chaîne dont le résultat diffère de son texte source.

```
<%# (7777000 * 7) %>
<%# "aa" + "bb" %>
```

Une réponse contenant `54439000` ou `aabb` confirme que le fragment a été évalué comme une expression, et non émis littéralement. À partir de là, passez à l'instanciation de types et aux appels de méthodes comme ci-dessus, en limitant les payloads aux namespaces que la page importe (`System`, et ce que l'application ajoute).

## Outils

- **Burp Suite** : Repeater pour injecter et confirmer les expressions de data binding compilées.
- Tests manuels avec Burp Repeater ; payloads façonnés par moteur.

## Références

- [Microsoft: Data-binding expression syntax](https://learn.microsoft.com/en-us/previous-versions/aspnet/ms178366(v=vs.100))
- [Microsoft: DataBinder.Eval method](https://learn.microsoft.com/en-us/dotnet/api/system.web.ui.databinder.eval)
- [OWASP: Expression Language Injection](https://owasp.org/www-community/vulnerabilities/Expression_Language_Injection)
