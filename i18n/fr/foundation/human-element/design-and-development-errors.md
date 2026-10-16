---
title: "Erreurs de conception et de développement : les faiblesses intégrées aux logiciels"
description: "Erreurs introduites lors de la construction des systèmes, d'une logique défaillante aux cas limites négligés, qui deviennent des faiblesses de sécurité bien avant que quiconque ne déploie le code."
keywords:
  - erreurs de conception et de développement
  - faiblesses logicielles
  - codage sécurisé
  - défauts de conception
  - validation des entrées
  - common weakness enumeration
---

# Erreurs de conception et de développement

Les **erreurs de conception et de développement** sont les faiblesses de sécurité intégrées aux logiciels pendant leur construction, avant que quiconque ne les installe, ne les configure ou n'en fasse un mauvais usage. Elles constituent la part de l'élément humain qui vit dans le code source et les schémas d'architecture : un développeur qui fait confiance à une entrée qui devrait être vérifiée, un architecte qui décide de stocker des mots de passe en clair, ou une équipe qui n'a jamais envisagé ce qui se passe lorsqu'un nombre devient plus grand que l'espace qui lui est réservé. Contrairement à un mauvais réglage, qu'un opérateur peut modifier, ces erreurs voyagent avec le logiciel lui-même, si bien que corriger l'une d'elles implique de livrer une nouvelle version.

## Défauts de conception contre bugs de codage

Deux erreurs très différentes se cachent sous la même étiquette, et il vaut la peine de les distinguer car elles se détectent et se corrigent de manières différentes.

| | Où elle se situe | Exemple | Comment elle se corrige |
|---|---|---|---|
| Défaut de conception | Dans le plan, avant que le code ne soit écrit | Faire confiance aux données du navigateur ; aucune authentification sur un point d'accès d'administration | Repenser l'architecture |
| Bug de codage | Dans la mise en œuvre d'un plan solide | Boucle avec erreur d'une unité ; une vérification de longueur manquante ; un test de valeur nulle oublié | Corriger la ligne précise |

Un défaut de conception peut passer toutes les revues de code parce que chaque ligne prise isolément est correcte ; l'erreur réside dans la forme du système, non dans la syntaxe. Un bug de codage est l'inverse : le plan était bon, mais une étape a été mise en œuvre de travers. Les deux reviennent au même pour un attaquant, une ouverture, et c'est pourquoi ils sont regroupés ici.

## Le catalogue de faiblesses de MITRE

Le secteur entretient un vocabulaire commun pour ces erreurs dans la Common Weakness Enumeration (CWE), un catalogue communautaire maintenu par MITRE. Une faiblesse est le type sous-jacent d'erreur, comme une validation des entrées incorrecte ou une écriture hors limites ; une vulnérabilité est une occurrence précise de cette faiblesse dans un produit réel. MITRE publie chaque année une liste des faiblesses logicielles les plus dangereuses, et les mêmes schémas la dominent année après année : injection, erreurs de gestion de la mémoire, et contrôles d'accès manquants ou défaillants. La répétition est le point essentiel. Il ne s'agit pas d'erreurs rares et ingénieuses mais de la même poignée d'oublis commis encore et encore.

## Décortiquer une vérification d'entrée manquée

Prenons un formulaire web qui recherche un client par numéro de compte.

1. **L'hypothèse.** Le développeur s'attend à un nombre et écrit du code qui fait confiance au champ pour en contenir un.
2. **La réalité.** Un utilisateur, ou un attaquant, saisit quelque chose qui n'est pas un simple nombre, comme des commandes de base de données ou une chaîne de caractères anormalement longue.
3. **L'erreur.** Parce que rien ne valide l'entrée avant qu'elle ne soit utilisée, ce texte est transmis directement à la base de données ou copié dans une zone de mémoire qui n'a jamais été dimensionnée pour lui.
4. **Le résultat.** Le système laisse fuiter des enregistrements, plante, ou exécute des instructions que le concepteur n'avait jamais prévues.

Rien ici n'a nécessité un exploit sophistiqué. La faiblesse était simplement l'absence d'une vérification qui aurait dû être présente, décidée au moment où le code a été écrit.

## Détecter les erreurs avant que le code ne soit livré

Parce que ces erreurs sont livrées à l'intérieur du produit, l'endroit le moins coûteux pour les éliminer est en amont. Réfléchir aux menaces pendant la conception, relire le code avec un second regard, et tester les schémas de faiblesses connus permettent tous de détecter les erreurs avant la mise en production ; c'est le même raisonnement qui sous-tend la [sécurité dès la conception](../principles/security-by-design.md). Ce qui passe entre les mailles devient le problème de l'équipe suivante : une configuration par défaut non sécurisée livrée dans le produit est elle-même une erreur de conception qui relève de cette page, tandis qu'un opérateur qui laisse plus tard ce réglage erroné crée une [mauvaise configuration](misconfiguration.md), et une interface déroutante pousse les gens vers les raccourcis traités dans [comportement et mauvais usage des utilisateurs](user-behavior-and-misuse.md). Les développeurs doivent aussi reconnaître un schéma dangereux pour éviter de l'écrire, et c'est pourquoi le codage sécurisé a sa place dans [formation et sensibilisation](training-and-awareness.md).

## Références

- [MITRE CWE: Common Weakness Enumeration](https://cwe.mitre.org/)
- [MITRE CWE Top 25 Most Dangerous Software Weaknesses](https://cwe.mitre.org/top25/)
- [OWASP Top Ten](https://owasp.org/www-project-top-ten/)
- [NIST SP 800-218, Secure Software Development Framework](https://csrc.nist.gov/pubs/sp/800/218/final)
- [OWASP Secure Coding Practices Quick Reference Guide](https://owasp.org/www-project-secure-coding-practices-quick-reference-guide/)
