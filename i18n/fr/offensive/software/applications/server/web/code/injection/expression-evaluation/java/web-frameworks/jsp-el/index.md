---
title: "Injection JSP EL : abus du langage d'expression de JavaServer Pages"
description: "JavaServer Pages évalue l'unified EL dans les expressions ${...}, et lorsque des données de requête atteignent une expression évaluée, cela expose des objets à portée pour le contournement d'autorisation et la réflexion pour l'exécution de code."
keywords:
  - injection JSP EL
  - EL de JavaServer Pages
  - unified EL
  - "expression ${}"
  - injection EL
---

# JSP EL

JavaServer Pages évalue l'unified expression language dans `${...}`, en résolvant les noms face aux portées page, requête, session et application au travers du `pageContext`. L'expression est évaluée immédiatement au rendu de la page. L'injection apparaît lorsque des données de requête atterrissent dans une expression que le conteneur évalue ensuite : un paramètre renvoyé dans un contexte `${...}`, une valeur passée à `application.evaluate` via l'`ExpressionFactory` JSP, ou un attribut de tag construit à partir d'une entrée utilisateur. Le conteneur exécute l'EL résultant avec les portées de la page à portée de main.

La surface atteignable correspond à la grammaire EL servlet. Les objets de portée implicites (`param`, `header`, `sessionScope`, `requestScope`, `applicationScope`, `pageContext`) exposent les attributs qu'une décision d'accès lit, et à partir d'EL 2.2 l'invocation de méthode atteint la réflexion et le runtime de l'hôte.

- **[Contournement d'autorisation](authorization-bypass.md)** : atteindre les maps de portée et le `pageContext` pour lire et forcer les valeurs dont dépend un contrôle.
- **[Exécution de code](code-execution.md)** : utiliser l'invocation de méthode et la réflexion pour atteindre un moteur de script ou `ProcessBuilder`.

## Outils

- **Burp Suite** : Repeater et Intruder pour injecter dans les expressions JSP ${...}.
- **J2EEScan** : extension Burp avec des contrôles d'injection Java EL.

## Références

- [Jakarta Expression Language Specification](https://jakarta.ee/specifications/expression-language/)
- [PayloadsAllTheThings: Java EL Injection](https://github.com/swisskyrepo/PayloadsAllTheThings)
