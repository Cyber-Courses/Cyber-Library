---
title: "Injection d'évaluation d'expression C#"
description: "Évaluateurs d'expressions et de formules .NET, des moteurs purement mathématiques comme NCalc au Flee compilant en IL, plus l'évaluation de data binding ASP.NET qui atteint le runtime de la page."
keywords:
  - injection d'expression C#
  - évaluateur d'expression .NET
  - NCalc
  - Flee
  - data binding ASP.NET
  - DataBinder.Eval
---

# C#

Les applications .NET recourent à un évaluateur d'expressions chaque fois qu'une formule, une règle ou un prix doit être configurable sans redéploiement. Ces évaluateurs couvrent une large plage d'impact. Un moteur de math et de logique confiné aux nombres et aux booléens est une cible différente d'un moteur qui compile en IL et peut appeler de vraies méthodes .NET, et une expression de data binding qui s'exécute dans le contexte de la page est encore différente.

La distinction qui décide du résultat est ce que le moteur peut atteindre. NCalc parse l'arithmétique, les comparaisons et les fonctions que l'hôte a enregistrées, et rien de plus par lui-même. Flee compile chaque expression en IL et invoque de vraies méthodes .NET sur les types que l'hôte a importés, donc son plafond est la surface de types importés. Le data binding ASP.NET évalue un chemin de propriété comme une expression .NET dans la page, donc le contrôle par l'attaquant de la chaîne d'expression (et pas simplement de la valeur liée) s'exécute dans le contexte de cette page.

## Moteurs

- **[Data binding ASP.NET](asp-net-data-binding.md)** : expressions `Eval`/`DataBinder.Eval` évaluées contre la page, où une chaîne d'expression contrôlée s'exécute comme une expression .NET.
- **[Flee](flee.md)** : le Fast Lightweight Expression Evaluator, qui compile en IL et peut atteindre les méthodes et propriétés statiques des types importés.
- **[NCalc](ncalc.md)** : un évaluateur mathématique et logique dont la surface injectable est la grammaire math/logique, les hooks de paramètres et de fonctions de l'hôte, et toute fonction personnalisée que l'application a enregistrée.

Chaque page cartographie le point d'entrée de parsing et d'évaluation du moteur, ce que sa grammaire atteint, et l'escalade réaliste d'une formule d'apparence anodine à l'impact maximal du moteur.

## Outils

- **Burp Suite** : Repeater et Intruder pour injecter des payloads de formule et de data binding.
- Tests manuels avec Burp Repeater ; payloads façonnés par moteur .NET.

## Références

- [OWASP: Expression Language Injection](https://owasp.org/www-community/vulnerabilities/Expression_Language_Injection)
- [OWASP WSTG: Testing for Code Injection](https://owasp.org/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/07-Input_Validation_Testing/11-Testing_for_Code_Injection)
