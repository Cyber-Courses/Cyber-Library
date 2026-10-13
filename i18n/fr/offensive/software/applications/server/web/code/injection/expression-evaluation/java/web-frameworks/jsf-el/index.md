---
title: "Injection JSF EL : abus du langage d'expression de JavaServer Faces"
description: "JavaServer Faces évalue l'unified EL dans les expressions #{...}, et lorsque des données de requête atteignent une expression évaluée, cela expose des objets à portée pour le contournement d'autorisation et la réflexion pour l'exécution de code."
keywords:
  - injection JSF EL
  - EL de JavaServer Faces
  - unified EL
  - "expression #{}"
  - injection EL
---

# JSF EL

JavaServer Faces (JSF) lie les attributs et les valeurs des composants avec l'unified expression language, écrit `#{...}`. La valeur est une expression différée : le framework l'évalue lorsque le composant est rendu ou lorsqu'une action liée se déclenche, en résolvant les noms face au contexte faces et aux maps d'attributs à portée. Lorsqu'un attaquant influence le texte qui devient une expression JSF, via un paramètre renvoyé dans un attribut de composant, un filtre de recherche rendu dans une valeur liée, ou un flux qui compile une chaîne utilisateur avec l'`ExpressionFactory` de l'application, cette évaluation exécute de l'EL choisi par l'attaquant au sein de la requête.

Deux impacts découlent de ce que la grammaire EL atteint. Les objets de portée implicites (`sessionScope`, `applicationScope`, `requestScope`, `facesContext`) permettent à une expression de lire et d'écraser les attributs dont dépend une décision d'accès. À partir d'EL 2.2, la grammaire invoque des méthodes, ce qui atteint la réflexion et le runtime de l'hôte.

- **[Contournement d'autorisation](authorization-bypass.md)** : atteindre les maps d'attributs à portée et le contexte faces pour lire et forcer les valeurs qu'une décision contrôle.
- **[Exécution de code](code-execution.md)** : utiliser l'invocation de méthode et la réflexion pour atteindre un moteur de script ou `ProcessBuilder`.

## Outils

- **Burp Suite** : Repeater et Intruder pour injecter dans les expressions JSF #{...}.
- **J2EEScan** : extension Burp avec des contrôles d'injection Java EL.

## Références

- [Jakarta Expression Language Specification](https://jakarta.ee/specifications/expression-language/)
- [PayloadsAllTheThings: Java EL Injection](https://github.com/swisskyrepo/PayloadsAllTheThings)
