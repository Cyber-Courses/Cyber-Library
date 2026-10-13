---
order: 7
title: "Injection d'évaluation d'expression"
description: "Lorsqu'une application évalue au runtime une chaîne d'expression influencée par l'attaquant, le langage d'expression devient un chemin de code. Organisé par langage et par moteur, des évaluateurs mathématiques aux interpréteurs EL complets qui atteignent le runtime de l'hôte."
keywords:
  - injection de langage d'expression
  - injection EL
  - SpEL
  - OGNL
  - évaluation d'expression au runtime
  - exécution de code
---

# Évaluation d'expression

L'injection d'évaluation d'expression se produit lorsque le code applicatif prend une chaîne d'entrée non fiable et la remet à un moteur d'expression qui l'évalue au runtime. Le moteur est censé calculer une formule, filtrer une règle ou résoudre une variable de template, mais lorsque la chaîne elle-même est contrôlée par l'attaquant, sa grammaire devient une surface d'exécution. Selon le moteur, cela va de l'évaluation purement arithmétique à l'accès complet au runtime de l'hôte, ce qui sur la JVM et dans les langages de scripting signifie l'exécution de code à distance.

Le sink est un appel qui compile et exécute une expression : un `SpelExpressionParser` Spring, une évaluation OGNL Struts, un `eval` PHP, un `eval`/`exec` Python, un moteur de règles résolvant une condition, ou une balise de data binding qui évalue un chemin de propriété. Chacun prend une chaîne et l'exécute, et chacun n'est pas plus sûr que le langage qu'il expose et le sandbox qui l'entoure.

## Organisé par langage et par moteur

Les sous-arbres se regroupent par le langage que l'application exécute, car les payloads atteignables et l'escalade vers l'exécution de code sont spécifiques au langage :

- **[C#](c-sharp/index.md)** : évaluateurs d'expressions et de formules .NET (NCalc, Flee) et évaluation de data binding ASP.NET.
- **[Go](go/index.md)** : le package d'expression `expr` et les mini-langages similaires.
- **[Java](java/index.md)** : la surface la plus riche, avec des interpréteurs EL complets (SpEL, OGNL, MVEL, JEXL), l'EL unifié de JSP et JSF, et des moteurs de règles (Drools, Camel, Camunda). La plupart atteignent `java.lang.Runtime` et donnent l'exécution de code.
- **[JavaScript](javascript/index.md)** : évaluation côté serveur et prototype pollution d'objets partagés.
- **[PHP](php/index.md)** : `eval` et chemins de code dynamiques.
- **[Python](python/index.md)** : mauvais usages d'`eval`/`exec` et langages de policy tels que CEL.

## Ce que le moteur décide

Deux propriétés du moteur décident du résultat. D'abord, ce que le langage peut atteindre : un évaluateur purement mathématique confiné aux nombres est une cible plus faible qu'une EL capable de nommer des classes arbitraires et d'appeler des méthodes. Ensuite, si un sandbox le contraint, et si ce sandbox peut être échappé, puisque plusieurs moteurs embarquent une restriction que la réflexion ou une référence de type contourne. Chaque page cartographie le point d'entrée de parsing et d'évaluation de son moteur, ce que la grammaire d'expression atteint, et la route d'une formule d'apparence anodine à l'impact maximal du moteur.

L'abus de moteurs d'expressions régulières (backtracking catastrophique, motifs contrôlés par l'utilisateur) est un sujet connexe mais distinct et vit dans ses propres pages inter-langages plutôt qu'ici, et le rendu de fichiers de template (SSTI) vit sous Template Engine.

## Outils

- **Burp Suite** : Repeater et Intruder pour injecter et itérer des payloads d'expression à travers les sinks.
- **J2EEScan** : extension Burp avec des contrôles actifs pour l'injection d'expression Java EL et OGNL.
- **tplmap** : détecte et exploite l'injection de code basée sur eval dans les sinks PHP, Python et JavaScript.

## Références

- [OWASP: Expression Language Injection](https://owasp.org/www-community/vulnerabilities/Expression_Language_Injection)
- [OWASP WSTG: Testing for Code Injection](https://owasp.org/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/07-Input_Validation_Testing/11-Testing_for_Code_Injection)
- [PayloadsAllTheThings: Server Side Template Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Server%20Side%20Template%20Injection)
