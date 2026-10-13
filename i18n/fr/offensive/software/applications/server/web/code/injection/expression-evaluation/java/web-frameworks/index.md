---
order: 1
title: "Injection de langage d'expression dans les frameworks web Java : SpEL, OGNL, MVEL et unified EL"
description: "Les frameworks web Java évaluent un langage d'expression issu des données de requête, de l'unified EL de JSP et JSF aux moteurs de framework SpEL, OGNL et MVEL qui atteignent le runtime de l'hôte."
keywords:
  - injection de langage d'expression
  - EL des frameworks web Java
  - injection SpEL
  - injection OGNL
  - unified EL
---

# Frameworks web

Les frameworks web Java s'appuient sur des langages d'expression pour déplacer les données entre la requête, le contrôleur et la vue. Un tag de page résout un chemin de propriété, un contrôleur mappe un paramètre de requête sur un graphe d'objets, une annotation nomme une règle à évaluer. Chacun de ces éléments est un moteur d'expression, et lorsque la chaîne qu'il évalue provient des données de requête, la grammaire devient une surface d'exécution à l'intérieur du processus de l'application.

Deux familles apparaissent ici. L'unified EL défini par la pile servlet et partagé par JSP et JSF (`${...}` en JSP, `#{...}` en JSF) a été conçu pour lire et écrire des attributs à portée, et à partir d'EL 2.2 il invoque des méthodes, ce qui atteint la réflexion et le runtime de l'hôte. Les moteurs de framework (SpEL dans Spring, OGNL dans Struts2 et l'ancien WebWork, MVEL dans plusieurs couches de binding et de règles) sont des langages objet complets : ils nomment des classes arbitraires, construisent des objets et appellent des méthodes, de sorte que la surface atteignable de base inclut déjà `java.lang.Runtime` et `ProcessBuilder`.

Le chemin d'une expression injectée vers l'impact est spécifique au moteur, c'est pourquoi les sous-arbres sont découpés par moteur :

- **[MVEL](mvel.md)** : un langage d'expression proche de Java avec un accès direct aux types Java, évalué via `MVEL.eval` ou une expression compilée.
- **[JSF EL](jsf-el/index.md)** : l'unified EL `#{...}` de JavaServer Faces, atteignant les objets à portée pour le [contournement d'autorisation](jsf-el/authorization-bypass.md) et la réflexion pour l'[exécution de code](jsf-el/code-execution.md).
- **[JSP EL](jsp-el/index.md)** : l'unified EL `${...}` de JavaServer Pages, couvrant le [contournement d'autorisation](jsp-el/authorization-bypass.md) et l'[exécution de code](jsp-el/code-execution.md).
- **[OGNL](ognl/index.md)** : l'Object-Graph Navigation Language de Struts2, couvrant une sonde d'évaluation de [listage de répertoire](ognl/directory-listing.md), l'[exécution de code à distance](ognl/remote-code-execution.md) et l'[inclusion de fichier distant](ognl/remote-file-inclusion.md).
- **[SpEL](spel/index.md)** : le Spring Expression Language, couvrant l'[exécution de code](spel/code-execution.md) et le [contournement de WAF](spel/waf-bypass.md).

## Outils

- **Burp Suite** : Repeater et Intruder pour injecter du SpEL, de l'OGNL, du MVEL et de l'unified EL.
- **J2EEScan** : extension Burp avec des contrôles actifs d'injection EL et OGNL.

## Références

- [OWASP: Expression Language Injection](https://owasp.org/www-community/vulnerabilities/Expression_Language_Injection)
- [PayloadsAllTheThings: Java EL Injection](https://github.com/swisskyrepo/PayloadsAllTheThings)
